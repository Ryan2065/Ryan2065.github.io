---
id: 713
title: 'CMMessaging Module Development'
date: 2018-09-25T01:10:39+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=713
permalink: /CMMessaging/
categories:
  - PowerShell
  - SCCM
---

# CMMessaging PowerShell Module v0.1

I've released a very early version of a module I started working on I'm calling [CMMessaging](https://www.powershellgallery.com/packages/CMMessaging). The goal of this project is to make some of the more advanced [Client Messaging SDK](https://www.nuget.org/packages/Microsoft.ConfigurationManagement.Messaging/) features easily available through PowerShell. What is the Messaging SDK? This project is by Microsoft and allows .Net developers to easily send messages to SCCM as clients or other servers. This has the ability to send DDRs, Hardware Inventory, Status Messages, and many other kinds of messages!

So, how to get started? First off you need the module! The easiest way to get the module is to install it from the PowerShell Gallery:

``` PowerShell
Install-Module CMMessaging
```

After the module is installed, your first task will always be to set the Client information. This will set the Management Point, Site Code, Client Certs, and any other initial information we'll need to send these messages. You can set it with this:

``` PowerShell
Set-CMMessagingClient
```

The above will set you up in local mode which will load all information from the local client. It will require admin rights. Every setting it gathers can be manually set with parameters in the function, so you can also do something like:

``` PowerShell
Set-CMMessagingClient -ClientName 'MyFakeClient' -CertFilePath 'C:\MyCert.pfx' -CertificatePassword ( ConvertTO-SecureString 'MyPassword' -AsPlainText -Force )
```

The above will set a fake client name and set it up with the specified certificate, but will look at the local computer environment to find Site Code, Management Point, and Domain.

After you run Set-CMMessagingClient, you can start running any of the other commands to send messages to your site server. They currently are:

### Register-CMMessagingClient

This will send a ConfigMgrRegistrationRequest to the site server.

### Send-CMMessagingDDR

This will send a DDR message

### Send-CMMessagingHardwareInventory

This will send a hardware inventory message, gathering the specified classes from the local machine. So if I run the command:

``` PowerShell
Set-CMMessagingClient
Send-CMMessagingHardwareInventory -Classes 'Win32_ComputerSystem'
```

it will load the data from the local system Win32_ComputerSystem and send it. This will be done immediately, bypass the inventory cycle, and be done even if Win32_ComputerSystem is not collected. To me, this is currently the most interesting set of commands as it allows us to send data to the CM server on demand through Hardware Inventory. The only limitation is the CM server needs to know how to process the class, so it has to have been discovered through Client Settings at least once.

And that's it! There's a reason it's v0.1!  Documentation on the Client Messaging SDK can be found [here](https://msdn.microsoft.com/en-us/library/mt744369.aspx). If you see anything in there you want added, please let me know!

The next feature I'm eyeing to add is support for sending Status Messages. After that, I'm open to requests!