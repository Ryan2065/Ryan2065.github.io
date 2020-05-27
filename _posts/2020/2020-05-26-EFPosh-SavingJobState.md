---
id: 721
title: 'EFPosh - Job State'
date: 2020-05-26
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=721
permalink: /EFPoshSavingJobState/
categories:
    - PowerShell
---

# EFPosh - Job State

For over a year I've been trying to put Entity Framework inside of PowerShell because of how powerful the toolset is in C#. I finally made headway a month ago and was able to create the PowerShell module [EFPosh](https://github.com/Ryan2065/EFPosh).

Entity Framework is hard to explain exactly how amazing it is, so I'm going to go through a few use cases to show how amazing this tool is!

## Job Tracker

One thing that's not the easiest to do in PowerShell is creating a job tracker. I was working recently on a migration job where we had distinct entities in one system, and wanted to move them to another. There were thousands of objects and each migration takes time to run, so the risk of this long running job being stopped was high. I wanted some easy way to lookup if an object was already processed, and skip it if it was.

### Building a Job Tracking Class

Getting started with Entity Framework is as easy as building a PowerShell class! First, we have to put some thought into this class, mainly we have to identify something called the Primary Key. The Primary Key is something that's unique and will always be unique. If I'm migrating SCCM collections, that's going to be the Collection Id. If I'm migrating Active Directory objects, it's the SID. The point is data you use usually has a primary key and you can just use the existing one. For my purposes, I'm migrating collections so I'll use the collection Id.

``` PowerShell
Class JobTracker {
    [string]$CollectionId
    [bool]$Complete
}
$Tables = @( New-EFPoshEntityDefinition -Type 'JobTracker' -PrimaryKeys 'CollectionId' )
$Context = New-EFPoshContext -SQLiteFile '.\JobTracker.sqlite' -Entities $Tables -EnsureCreated
```

The above code will create a Sqlite database in the current directory called JobTracker with a table JobTracker!

To create a database, all Entity Framework needs is a class describing each table, and then tell it what kind of database to create. First I create a $Tables array saying we want to create a table of type JobTracker with CollectionId as a primary key. Then, I create the context with the -EnsureCreated switch, which will create the database if it does not exist!

> The Context (output from New-EFPoshContext) is required for everything in Entity Framework. You can keep it in a variable or re-create it as needed.

Now, the details of how we get collections to migrate doesn't matter for this blog, so let's handwave it and get all those collections!

``` PowerShell
$CollectionsToMigrate = Get-CollectionsToMigrate
```

After getting the collections, we'll skeleton out a loop:

``` PowerShell
Foreach($Collection in $CollectionsToMigrate) {

}
```

Now, first we want to pull up the record for this collection:

``` PowerShell
    New-EFPoshQuery -DBContext $Context -Entity JobTracker
    Add-EFPoshQuery -Property CollectionId -Equals $Collection.CollectionId
    $Record = Start-EFPoshQuery -FirstOrDefault
```

Entity Framework uses Linq to build queries, and EFPosh continues that. The first line tells EFPosh you want to create a new query against Table JobTracker. The second line tells it you want to search Column CollectionId and find any rows that equal $Collection.CollectionId. The last line tells EFPosh to run the query and only return the first result or $null if there are no results. Using EFPosh, you can build up your query with multiple Add-EFPoshQuery commands.

After we get the record, we should look to see if it contains anything. If it did not find a record, create one to track progress:

``` PowerShell
if($null -eq $Record){
    $Record = [JobTracker]::new()
    $Record.CollectionId = $Collection.CollectionId
    $Context.Add($Record)
    $Context.SaveChanges()
}
```

To add a new record to the database, you simply create a new object of type JobTracker, and fill out all the information. In this case, we only have to fill out CollectionId. Then, add it to the context, and then save the changes.

Now that we have a $Record object (either from the database or newly created), let's evaluate it:

``` PowerShell
if($false -eq $Record.Complete){
    Write-Host "Migrating $($Collection.CollectionId)"
    Start-CollectionMigration -Collection $Collection
    $Record.Complete = Test-CollectionMigration -Collection $Collection
    $Context.SaveChanges()
}
```

Here, we say if the Record is not complete, start the migration. After running the migration, test if it was complete with Test-CollectionMigration. Store the results in ```$Record.Complete``` and run ```$Context.SaveChanges()``` to save the results! Entity Framework has a Change Tracker that runs in the background. Any changes to entities are automagically tracked and when you call SaveChanges() are immediately written to the database.

And that's it! We are now saving the job status to the database, so if we have to stop the job we can easily resume, or if there's some bug in our code and only 90% of collections migrated, we can fix the bug and only resume the 10% that didn't migrate!

How cool was that? This blog showed you how to create a database, query, add, and modify data all without writing ANY SQL!

You can see a [gist](https://gist.github.com/Ryan2065/436d851fc2d45d3804db7ca0d2057fa3) of the entire script here. I've included fake functions for the "Collection Migration" pieces. To follow with the blog, start on line 19.

<script src="https://gist.github.com/Ryan2065/436d851fc2d45d3804db7ca0d2057fa3.js"></script>
