---
id: 389
title: Change Driver Source Directories
date: 2014-07-10T20:09:49+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=389
permalink: /change-driver-source-directories/
categories:
  - ConfigMgr
  - Powershell
---
Don’t change the name of the driver source directory. Don’t ever do this.

If you decided to do this, here is a script which will help you. It will find all drivers whose source path starts with $OldPath. It will then replace the $OldPath with the $NewPath.  Of course, you need to specify the $SiteCode and $Server.
<pre class="lang:ps decode:true ">#Edit these four lines
$SiteCode = "PS1"
$Server = "CM12"
$OldPath = "\cm12sourceOSDDrivers"
$NewPath = "\cm12sourceOSDDriver Source"
 
#Don't edit below this line
$FindPath = $OldPath.Replace("","\")
Get-WmiObject -Namespace "rootsmssite_$SiteCode" -ComputerName $Server -Query "select * from SMS_Driver where ContentSourcePath like '$FindPath%'" | ForEach-Object {
	$ContentSource = $_.ContentSourcePath
	$ContentSource = [Regex]::Escape($ContentSource) -replace [Regex]::Escape($OldPath),[Regex]::Escape($NewPath)
	$_.ContentSourcePath = [regex]::Unescape($ContentSOurce)
	$_.Put()
}</pre>