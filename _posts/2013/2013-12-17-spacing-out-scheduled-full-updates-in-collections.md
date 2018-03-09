---
id: 147
title: Spacing Out Scheduled Full Updates In Collections
date: 2013-12-17T20:33:38+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=147
permalink: /spacing-out-scheduled-full-updates-in-collections/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Collections
  - Schedule
---
Here is a quick script to change a collection’s scheduled update to a random time. You can use this if you decide you want to randomize collection updates.

This script will pick a random time in the next week (or sooner / later if you change day span) and set the collection to update then. It will also change the refresh schedule to either 2 (No incremental updates) or 6 (incremental updates) depending on what you want in your environment. Remember, you shouldn’t have any more than 200 collections with incremental updates. I’d suggest you keep the number as low as possible.

Variables To Change:

$CollectionName: Put in the name of the collection to change. % is a wildcard to make it change more than one.
$ServerName: The name of your CM12 server
$SiteCode: The site code of your CM site
$DaySpan: How often the collection should refresh. By default it is set to 7 days. Change it if you want
$RefreshType: Set it to 2 if you want incremental updates off, or 6 if you want incremental updates enabled

Script:
 
{% highlight powershell linenos %}
<#
By Ryan Ephgrave
#>
$CollectionName = "%"   #Use % for a wildcard - If it's only % it will change all collections
$ServerName = "CM12"	#ConfigMgr Server
$SiteCode = "PS1"		#Site Code
$DaySpan = 7			#How often should the update happen
$RefreshType = 2		#6 - Enables incremental updates / 2 - Disables incremental updates
 
$Collections = Get-WmiObject -Namespace "root\sms\site_$SiteCode" -ComputerName "$ServerName" -query "Select * from SMS_Collection where Name like '$CollectionName'"
Foreach ($Collection in $Collections) {
    $ColID = $Collection.CollectionID
    if (!($ColID.StartsWith("SMS"))) {
 	   $AddDays = Get-Random -Maximum $DaySpan
    	$AddHours = Get-Random -Maximum 24
	    $DateTime = (Get-Date).AddDays($AddDays).AddHours($AddHours)
    	$WMIDateFormat = Get-Date $DateTime -Format yyyyMMddHHmmss.000000+***
         $WMIClassPath = "\\$ServerName\root\sms\site_$SiteCode" + ":SMS_ST_RecurInterval"
	    $WMIClass = [WMIClass]($WMICLassPath)
    	$NewColRefreshSch = $WMIClass.CreateInstance()
	    $NewColRefreshSch.DaySpan = $DaySpan
    	$NewColRefreshSch.StartTime = $WMIDateFormat
    	$Collection.Get()
	    $Collection.RefreshType = $RefreshType
    	$Collection.RefreshSchedule = $NewColRefreshSch
	    $Collection.Put()
    }
}</pre> 
{% endhighlight %}
