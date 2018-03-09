---
id: 39
title: Automagically Clear Expired Updates From Software Update Groups
date: 2015-05-21T22:49:31+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=39
permalink: /automagically-clear-expired-updates-from-software-update-groups/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Software Updates
  - Status Message
---
I’ve seen a number of right click tools out there that remove expired updates from SUGs. Seeing as I’m so against right click tools, I thought I’d figure out a better way to do this!

First off, I need a way to remove expired updates from SUGs. For that I turn to VBS! Just kidding, it’s in Powershell… Who still uses VBS for their ConfigMgr automation?

Anyway, here is the script:
<pre class="toolbar:2 lang:ps decode:true">$SiteCode = 'PS1'
$AllExpiredUpdates = Get-WmiObject -Namespace "root\sms\site_$SiteCode" -Query "Select SoftUp.CI_ID From SMS_SoftwareUpdate as SoftUp join SMS_CIRelation as Rel on SoftUp.CI_ID=Rel.ToCIID where SoftUp.IsExpired='True' AND Rel.RelationType='1'"
 
Get-WmiObject -Namespace "root\sms\site_$SiteCode" -Query 'select * from SMS_AuthorizationList where AssociatedAutoRuleID IS NULL' | ForEach-Object {
    $UpdatesList = New-Object -TypeName System.Collections.ArrayList
    $_.Get()
    $UpdateCount = $_.Updates.Count
        
    Foreach ($Update in $_.Updates) {
        $NotFound = $true
        Foreach ($instance in $AllExpiredUpdates) {
            If ($instance.CI_ID -eq $Update) {
                $NotFound = $false
            }
        }
        If ($NotFound) { $UpdatesList.Add($Update) | Out-Null }
    }
 
    If ($UpdateCount -gt $UpdatesList.count) {
        $_.Updates = $UpdatesList
        $_.Put()
    }
}</pre>
Change $SiteCode to your site code and it’s good! This first finds all updates that are expired AND in a software update group. It then cycles through all groups not created by an ADR, finds all updates in the group, and compares them to the expired list. If one is expired, it removes it.

Now, I need to make this automatically run so I can avoid having to manually trigger it. Taking all that time to click two buttons is hard…

I recently have been playing around with status messages, so I thought I’d give status filter rules a shot. I found when WSUS successfully finishes syncing, it sends a status message ID of 6702. Using this information, I created a status filter rule to run my script whenever WSUS finishes syncing.

To create this status filter rule, go to Administration -&gt; Site Configuration -&gt; Sites and then right click on your site(s). Now, select the option you’ve probably never used before, Status Filter Rules:
<p id="WXWPZwK"><img class="alignnone size-full wp-image-54 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb05263dab.png" alt="" /></p>
The list will pop up and click Create…
<p id="hmlzwyA"><img class="alignnone size-full wp-image-55 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb0665c8ca.png" alt="" /></p>
Name your rule and then select Message ID: 6702
<p id="fJwrnfZ"><img class="alignnone size-full wp-image-57 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb08998783.png" alt="" /></p>
use this command line:

C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Bypass -File "C:\Scripts\Remove-ExpiredUpdates.ps1"

Otherwise, change it to point to the ps1 file you created in the first step.
<p id="fIYetEo"><img class="alignnone size-full wp-image-58 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb0a419f72.png" alt="" /></p>
And you’re done! Click Next Next Finish and then kick off a WSUS sync. After it’s done you’ll see a Powershell.exe process start from User System, and then 30-60 seconds later all your software update groups will have no expired updates in them!

&nbsp;