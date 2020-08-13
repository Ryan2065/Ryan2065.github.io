---
id: 722
title: 'EFPosh - How I use it'
date: 2020-08-10
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=722
permalink: /EFPoshHowIUseIt/
categories:
    - PowerShell
---

# How I use EFPosh

I developed EFPosh just a few months ago to make interacting with SQL easy from PowerShell, but the tooling has evolved a lot the more I use it at work. I thought it'd be interesting to write how I use this PowerShell module in my day-to-day work to give people their own ideas.

## Database Context

The idea behind a database context is you build out the database schema in Entity Framework first, then you can query the DB, edit data, or even build the database from scratch. In working with EFPosh, I've gravitated towards building a database context file, and then just loading the file in my scripts. EFPosh has the ability (and I use it all the time) to build out your context for you!

``` PowerShell
$SplattedParams = @{
    'MSSQLServer' = 'Lab-CM.Home.Lab'
    'MSSQLDatabase' = 'CM_PS1'
    'MSSQLIntegratedSecurity' = $true
    'FilePath' = "$($env:Temp)\CMContext.ps1"
    'Overwrite' = $true
    'EntitesToMap' = @(
        'v_Collections',
        'v_R_System'
    )
}
Start-EFPoshModel @SplattedParams

```

The above code will create a DBContext file CMContext.ps1 with parameters for Server, Database, and Credential and has all the plumbing to query the views v_Collection and v_R_System. 

At work, I'll generally wrap the above in a function like this to handle the creation if it doesn't exist:

``` PowerShell
Function Get-CMContext {
    Param(
        $Server,
        $Database
    )
    $ContextFile = "$($env:Temp)\CMContext.ps1"
    if(-not ( Test-Path $ContextFile )){
        $SplattedParams = @{
            'MSSQLServer' = 'Lab-CM.Home.Lab'
            'MSSQLDatabase' = 'CM_PS1'
            'MSSQLIntegratedSecurity' = $true
            'FilePath' = $ContextFile
            'Overwrite' = $true
            'EntitesToMap' = @(
                'v_Collections',
                'v_R_System'
            )
        }
        Start-EFPoshModel @SplattedParams
    }
    return . $ContextFile -Server $Server -Database $Database
}
```

## Querying data

Once I have the context, the rest is a breeze! I'll re-use the function from above to get the context and then query for a collection:

``` PowerShell
$Context = Get-CMContext -Server 'Lab-CM.Home.Lab' -Database 'CM_PS1'
Search-EFPosh -Entity $Context.v_Collections -Expression { $_.CollectionName -eq 'MyNewDeviceCollection' }
```

In Entity Framework, entities are classes that map to database objects. If I open up my CMContext.ps1 file, I can see the entity v_Collections:

``` PowerShell
Class v_Collections {
    [int] $CollectionID
    [string] $SiteID
    [string] $CollectionName
    [string] $LimitToCollectionID
    [string] $LimitToCollectionName
}
```

Each property on this class corresponds to a column on the SQL view. I can edit this class and remove properties to limit what's brought back from SQL, or I can leave it as is. For the sake of space, I edited the class to only include properties I care about. I highly recommend you edit these and only bring back the columns you care about.

Back to the searching example!

``` PowerShell
Search-EFPosh -Entity $Context.v_Collections -Expression { $_.CollectionName -eq 'MyNewDeviceCollection' }
```

So now that I know what entities are, we can see above I'm telling EFPosh to query the view v_Collections for the context $Context (which has the server/connection info). Then, I'm telling it to filter v_Collections and only return the collections that have a CollectionName of MyNewDeviceCollection.

Run it and you get great output

``` PowerShell
PS C:\Users\Ryan> Search-EFPosh -Entity $Context.v_Collections -Expression { $_.CollectionName -eq 'MyNewDeviceCollection' }


CollectionID          : 16777229
SiteID                : PS100014
CollectionName        : MyNewDeviceCollection
LimitToCollectionID   : SMS00001
LimitToCollectionName : All Systems
```

What is happening on the backend? There's an "easter egg" (undocumented) feature that lets you see what Entity Framework does in the background. Create an environment variable called EFPoshLog and set it to 'true', then re-create the DB context. 

``` PowerShell
$env:EFPoshLog= 'true'
$Context = Get-CMContext -Server 'Lab-CM.Home.Lab' -Database 'CM_PS1'
Search-EFPosh -Entity $Context.v_Collections -Expression { $_.CollectionName -eq 'MyNewDeviceCollection' }

info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core 2.2.6-servicing-10079 initialized 'PoshContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info
: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (5ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT [p].[CollectionID], [p].[CollectionName], [p].[LimitToCollectionID], [p].[LimitToCollectionName], [p].[SiteID]
      FROM [dbo].[v_Collections] AS [p]
      WHERE [p].[CollectionName] = N'MyNewDeviceCollection'

CollectionID          : 16777229
SiteID                : PS100014
CollectionName        : MyNewDeviceCollection
LimitToCollectionID   : SMS00001
LimitToCollectionName : All Systems
```

What happened is EFPosh takes your PowerShell binary expression, translates it to a Linq binary expression, and then Entity Framework translates that to a SQL query. If you think this might be prone to errors, yeah. It's not perfect. Because of this the only supported expressions right now are ones that have a left and right side of the expression (ie, ```$_ -eq 5```). Expressions that simply run methods are unsupported (ie, ```$_.Equals(5)```).

One really cool thing about entity framework is the ability to create complex logic with little effort. One function I use a lot at work is getting the children of a parent collection recursively. There's not a lot to the function, so let's just put it out there:

``` PowerShell
Function Get-RecursiveCollections{
    Param(
        [string[]]$CollectionNames
    )
    $Results = Search-EFPosh -Entity $Context.v_Collections -Expression { $0 -contains $_.LimitToCollectionName } -Arguments @(,$CollectionNames)
    if($Results.Count -gt 0){
        Get-RecursiveCollections -CollectionNames $Results.CollectionName
    }
    $Results
}
```

This function calls Search-EFPosh and passes the expression ```{ $0 -contains $_.LimitToCollectionName }```. What in the world is $0? This corresponds to index 0 in the Arguments array, so we also have ```-Arguments @(,$CollectionNames)```. Before you mosey on down to the comments to talk about the "misplaced" comma, that's supposed to be there! I'm not entirely sure what it does, but without it PowerShell throws away @() for single-object arrays and leaves you with just $CollectionNames, so we'd be indexing into $CollectionNames, which is not what we want!

So what does $0 -contains $_.LimitToCollectionName turn into? Here's the generated SQL!

```
      SELECT [p].[CollectionID], [p].[CollectionName], [p].[LimitToCollectionID], [p].[LimitToCollectionName], [p].[SiteID]
      FROM [dbo].[v_Collections] AS [p]
      WHERE [p].[LimitToCollectionName] IN (N'Collection 1')
```

So when we say "Array" -contains "propertyName" it turns into the SQL equivalent of Column IN ().

Back to our function, I then check if there are any results, and if there are, call this function again to get the children! Lastly, return the results.

With 5 minutes of programming, I was able to create a function that dynamically generates SQL queries to recursively pull back data from SQL!

So that's a little bit of how I use EFPosh at work, from context management to using that context, it's all pretty easy!

