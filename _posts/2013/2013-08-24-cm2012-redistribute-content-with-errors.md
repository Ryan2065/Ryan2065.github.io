---
id: 96
title: CM2012 – Redistribute content with errors
date: 2013-08-24T15:19:11+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=96
permalink: /cm2012-redistribute-content-with-errors/
categories:
  - ConfigMgr
  - Powershell
tags:
  - ConfigMgr
  - Content
  - Powershell
  - System Center Configuration Manager
---
Here’s a quick script that will redistribute all content that has errors to the DPs it errored out on.
<pre class="lang:ps decode:true ">$Server = "CM2012"
$Namespace = "root\sms\site_PS1"
$strQuery = "Select Name,PackageID from SMS_DistributionDPStatus where MessageState &gt; 2"
Get-WmiObject -Query $strQuery -Namespace $Namespace -ComputerName $Server | ForEach-Object {
    $ServerName = $_.Name
	$ServerName = $ServerName.ToUpper()
	$PackageID = $_.PackageID
	$strQuery = "select * from SMS_DistributionPoint where PackageID = '$PackageID'"
	$PackageDPGroup = Get-WmiObject -Query $strQuery -Namespace $Namespace -ComputerName $Server
	foreach ($PackageDP in $PackageDPGroup) {
		$NalPath = $PackageDP.ServerNalPath
		if ($NalPath.ToUpper().Contains("$ServerName")) {
			$Error.Clear()
			$PackageDP.RefreshNow = $true
			$PackageDP.Put()
		}
	}
}</pre>
Just run this, and in a few minutes you’ll see your content start to redistribute on the DPs it errored on.