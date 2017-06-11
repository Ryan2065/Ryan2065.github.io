---
id: 627
title: How to use PowerShell to start ConfigMgr updates in current branch
date: 2016-07-31T14:57:34+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=627
permalink: /use-powershell-start-configmgr-updates-current-branch/
categories:
  - ConfigMgr
  - Powershell
tags:
  - ConfigMgr
  - Current branch
  - hotfix
  - Powershell
  - SCCM
---
I’m one of those people who rebuilds their test lab often. I like to tinker with WMI and all sorts of other things, so I rebuild to wipe all my configuration changes and start fresh. Current Branch has put a wrinkle in this because I could never start the updates automatically with my build script. Well, I’ve figured out how to start the SCCM updates with a script!

 
<pre class="lang:ps decode:true " >$ParamHash = @{
    NameSpace = 'root\sms\site_ps1'
    Query = 'Select * from SMS_CM_UpdatePackages where Name like "Configuration Manager 1602 Hotfix (KB3174008)"'
}
$Update = Get-WmiObject @ParamHash
$Update.UpdatePrereqAndStateFlags(0,2)</pre> 


The above code will start the KB3174008 update:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-62.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-62.png" alt="image" width="644" height="142" border="0" /></a>

You can now use the above code in your scripts to start the updates! I’ll be working on my SCCM build script and will find a good way to wait for updates to arrive and then install them. Once that’s complete, I’ll post it here!