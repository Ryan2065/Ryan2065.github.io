---
id: 306
title: 'ConfigMgr: Add multiple users to a collection'
date: 2013-10-07T19:24:03+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=306
permalink: /configmgr-add-multiple-users-to-a-collection/
categories:
  - ConfigMgr
  - Powershell
---
<p>A user at myitforum asked if there was a way to add multiple users to a collection. I wrote a script for the new right click tools to do this, but since that one isn’t released yet I decided to quickly re-write it to accept a file as the input. <p>Usage: Supply the collection name, file name of the users, server, and site code. The file needs to have the unique username of each user, so domainusername. If you aren’t sure, go to the properties of one of your users in ConfigMgr and look at the unique username field for the proper formatting.
 
<pre class="lang:ps decode:true " >$Server = "SERVER"
$SiteCode = "SITE CODE"
$UserList = "FileName"
$ColName = "Collection Name"
##Don't edit below here##
 
$Namespace = "root\SMS\site_$SiteCode"
$strQuery = "Select CollectionID from SMS_Collection where Name like '$ColName'"
$ColID = (Get-WmiObject -Query $strQuery -Namespace $NameSpace -ComputerName $Server).CollectionID
if ($ColID -ne $null) {
	Get-Content $UserList | ForEach-Object {
		$Username = $_
		$Username = $Username.replace("\","_")
		$strQuery = "Select ResourceID from SMS_R_User where UniqueUserName like '$UserName'"
		$ResourceID = (Get-WmiObject -Query $strQuery -Namespace $Namespace -ComputerName $Server).ResourceID
		if ($ResourceID -ne $null) {
			$Collection=[WMI]"\\$($Server)\$($Namespace):SMS_Collection.CollectionID='$ColID'"
			$RuleClass = [wmiclass]"\\$($Server)\$($NameSpace):SMS_CollectionRuleDirect"
			$newRule = $ruleClass.CreateInstance()
			$newRule.RuleName = $RuleName
			$newRule.ResourceClassName = "SMS_R_User"
			$newRule.ResourceID = $ResourceID
			$Collection.AddMembershipRule($newRule)
		}
	}
}</pre> 
