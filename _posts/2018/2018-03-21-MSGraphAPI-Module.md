---
id: 712
title: 'MSGraphAPI PowerShell Module'
date: 2018-03-21T01:10:39+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=712
permalink: /MSGraphAPI-Module/
categories:
  - PowerShell
  - Graph
---

# MSGraphAPI PowerShell Module

I've developed a PowerShell module for interacting with Microsoft Graph, and it's in beta! 

The module [MSGraphAPI](https://www.powershellgallery.com/packages/MSGraphAPI) is available 
through the PowerShell Gallery so you can download it on any PowerShell 5+ machine with the 
command

```
Install-Module MSGraphAPI
```

The module will only expose two commands, but has a LOT of features! First, to connect you'll 
need to use an Azure application registration. Let's quickly look at how we do that. If you 
already have an application registration, skip to the "Let's Begin" section

## Create an application registration
Go to the [Azure Portal](https://portal.azure.com) and click the link for Azure
Actice Directory, then go to App Registrations:

![AppRegistrations](\images\2018-03-21\EphingAdmin-0004.jpg)

Now click "New application registration" at the top:

![NewAppRegistrations](\images\2018-03-21\EphingAdmin-0005.jpg)

Now you'll want to fill out Name, Application Type, and Redirect URI. Here's what 
I have:

```
Name: MSGraph API
Application Type: Native
Redirect URI: urn:ietf:wg:oauth:2.0:oob
```
![NewAppRegistrationInfo](\images\2018-03-21\EphingAdmin-0050.jpg)

Note, for the redirect URI, the code `urn:ietf:wg:oauth:2.0:oob` tells Microsoft
to return the authorization code in the HTML. It's kind of like having 
`http://localhost` and just knowing that's going to redirect you to the current 
machine.

Now, click Create and you should have a new application! You'll have one last 
thing to do - add permissions to your application registration. Click Settings:

![ClickSettings](\images\2018-03-21\EphingAdmin-0051.jpg)

Now click Required Permissions, and Add:

![RequiredPermissionsAdd](\images\2018-03-21\EphingAdmin-0052.jpg)

Click Select an API, then select Microsoft Graph:

![SelectAPIGraph](\images\2018-03-21\EphingAdmin-0053.jpg)

Select any Delegated Permissions you think you need! MSGraph API currently only 
supports Delegated Permissions, which means both the application and the user 
requires permissions. I will be adding more authentication scenarios in the future,
but this is the only support one at the moment. If you followed my instructions
exactly, you will only see Delegated Permissions, because application registrations
of type "Native App" only allow for delegated permissions.

Check what permissions you'll need then click Select at the bottom:

![Check](\images\2018-03-21\EphingAdmin-0054.jpg)

Click Done at the bottom and now the application has those permissions! If you want, you can now
also click "Grant Permissions" to grant these permissions to your entire organization. 

# Let's Begin

First, you must get a token that the MSGraphAPI Module can use. You'll need 
the application information and a Azure AD credential. Here's how you connect:

```
$clientId = "9b0eacbf-09b7-4df9-bec8-55d1dacb7075" #In Azure AD, the Application ID
$redirectUri = "urn:ietf:wg:oauth:2.0:oob"
$resourceAppIdURI = "https://graph.microsoft.com" #What am I connecting to? Should always be Graph
$Tenant = "xxx.onmicrosoft.com"
$Credential = Get-Credential   # Give the Azure AD credentials
Connect-MSGraphAPI -Tenant $Tenant -ClientId $clientId -RedirectURI $redirectUri -Resource $resourceAppIdURI -Credential $Credential
```

If this is your first time running Connect-MSGraphAPI with that specific ClientId,
you may be prompted to give the Application rights to run in your Azure AD 
environment. You will need to give it the permissions it asks for in order to 
continue. If you are not comfortable giving it the requested permissions, follow 
the previous section and create a new application registration and give it less 
permissions.

Here's the screen you may see requesting permissions:

![RequestingPermissions](\images\2018-03-21\EphingAdmin-0056.jpg)

Now you are connected! Let's try getting information from Graph:

```
$Users = Invoke-MSGraphMethod -query 'Users'
```

When I look at `$Users[1]` I see that it returned some good information!

![PoshReturned](\images\2018-03-21\EphingAdmin-0055.jpg)

When I look at the type returned, I can see it is a MSGraphAPI_v1_user:

![UserType](\images\2018-03-21\EphingAdmin-0057.jpg)

This means the Module returned a custom class that contains all the properties 
already in the correct type so you don't have to do the conversion from string to
whatever type it needs to be in.  

The custom class returned should also have methods available for all Graph v1.0 
navigation properties, actions, and functions. If I want to get this user's 
registered devices, I simply run `$Users[1].Get_RegisteredDevices()`. If I want 
to assign a license to the user, just run 
`$Users[1].assignLicense($License, $null)`. 

And that's all there is to the MSGraph API Module, for now. The classes were 
automatically generated based off Graph's metadata, so anything returned should 
have a custom class that helps you perform all actions.

The module is currently in beta and I'm hoping people try it out and give me 
feedback at the [GitHub Repo](https://github.com/Ryan2065/MSGraphCmdlets). Feel
free to add any features you want also!

