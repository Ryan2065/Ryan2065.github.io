---
id: 62
title: Compliance Item – Cleanup IIS Logs
date: 2015-03-10T23:19:16+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=62
permalink: /compliance-item-cleanup-iis-logs/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Compliance
  - IIS
---
I know I know, another cleanup IIS log file post… Nash Pherson wanted to know if there was a way to make a CI which would search for all IIS Sites, get the log folder location for each site, and then cleanup the IIS logs. I looked into it and was able to figure it out. I like this process because you set up a CI and a collection and then it’s on auto pilot. You never have to look at it again, assuming the ConfigMgr client is on all your ConfigMgr servers.
<p style="text-align: center;">Configuration Item</p>
Make up a fun, unique name (I picked IIS Log File Cleanup) and then pick your supported platforms. I have tested this process on Server 2012 and Server 2012 R2, so if you have any Server 2008 systems you’ll want to test it out on them. Select new setting and then make your settings match this:
<p id="MlIqKxC"><img class="alignnone wp-image-63 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb4bd3cdc3.png" alt="" width="663" height="183" /></p>
<p style="text-align: center;">Discovery Script:</p>

<pre class="toolbar:2 lang:ps decode:true">$LogFileCount = 0
$DaysToKeep = -7
try {
    Import-Module WebAdministration
    Get-Website | ForEach-Object { 
        $LogDirectory = $_.LogFile.Directory
        If ($LogDirectory -match "(%.*%)\\") {
            $LogDirectory = $LogDirectory -replace "%(.*%)\\","$(cmd /c echo $matches[0])"
            $LogFileCount = $LogFileCount + (Get-ChildItem $LogDirectory -Recurse -Filter "*.log" | Where LastWriteTime -lt ((Get-Date).AddDays($DaysToKeep))).Count
        }
    }
    return $LogFileCount
}
catch { return -1 }</pre>
This imports the IIS Cmdlets, searches for all websites, does some fun regex stuff <a href="http://calumpowell.com/2013/02/17/using-powershell-to-delete-iis-log-files/" target="_blank">(thanks to this blog for the code to do it)</a> to get the log folder path, and then counts the number of log files modified before 7 days ago. You can adjust the 7 days by changing line 2 to negative whatever (another common one would be -14 for 2 weeks). The script will return -1 if there is an error.
<p style="text-align: center;">Remediation Script</p>

<pre class="toolbar:2 lang:ps decode:true ">$DaysToKeep = -7
try {
    Import-Module WebAdministration
    Get-Website | ForEach-Object { 
        $LogDirectory = $_.LogFile.Directory
        If ($LogDirectory -match "(%.*%)\\") {
            $LogDirectory = $LogDirectory -replace "%(.*%)\\","$(cmd /c echo $matches[0])"
            Get-ChildItem $LogDirectory -Recurse -Filter "*.log" | Where LastWriteTime -lt ((Get-Date).AddDays($DaysToKeep)) | Foreach-Object { Remove-Item $_.FullName -Force }
        }
    }
}
catch {  }</pre>
This does the same as the discovery script, except deleting the files. If you changed the days variable in the discovery script, make sure to change it in the remediation script also.
<p style="text-align: center;">Compliance Rules</p>
Compliance should be set to this:
<p id="ZbSzdiG"><img class="alignnone  wp-image-64 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb5be68149.png" alt="" width="621" height="266" /></p>
Then Next, Next, Finish.
<p style="text-align: center;">Configuration Baseline</p>
This one is simple. Simply create the baseline and then add the CI you just created. After it is created you can deploy it to a collection with all your ConfigMgr servers that have IIS. Make sure you select to Remediate and Remediate outside of of maintenance windows if you want to do that.

Here is a query you can use on a collection to find all those servers which are ConfigMgr and have IIS installed:

select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_SERVER_FEATURE on SMS_G_System_SERVER_FEATURE.ResourceId = SMS_R_System.ResourceId where SMS_G_System_SERVER_FEATURE.Name like “IIS%” AND ResourceNames[0] In (Select Distinct ServerName FROM SMS_SystemResourceList)

And that’s it! Now your IIS cleanup for ConfigMgr servers is on auto pilot.