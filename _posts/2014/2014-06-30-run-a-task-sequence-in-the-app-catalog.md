---
id: 386
title: Run a Task Sequence in the App Catalog
date: 2014-06-30T20:07:51+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=386
permalink: /run-a-task-sequence-in-the-app-catalog/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Task Sequence
---
I was talking with a co-worker earlier today and he said he was looking for a way to start a task sequence through the application catalog, but not have the task sequence show up in Software Center. He thought there was probably a way to do it by scheduling the task sequence far out into the future, but wasn’t exactly sure how to go about doing it. I’ve played around with starting task sequences through the right click tools, so I decided to take a look at it.

First off, why would you want to deploy a task sequence to the application catalog. I’ve meet a number of administrators who want to give their helpdesk the ability to reimage computers, but don’t like the idea of giving them the ability to deploy the task sequence or add computers to a collection with the task sequence deployed to it. In this case, you can have them open up the application catalog and trigger an install from there, and you have the power of determining which computers they can image and restrict their rights in the console if you wanted to.

Now, the first thing you will need to do is set up the deployment. I created a test machine collection and then deployed the task sequence as available to ConfigMgr clients. In production, you could create a collection with all workstations in it and deploy the task sequence there, but let’s try a test collection first!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-2-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image (2)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-2_thumb.png" alt="image (2)" width="417" height="233" border="0" /></a>

Schedule the deployment to be available far into the future, I set it for 2033 myself.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image1-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image1 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image1-1_thumb.png" alt="image1 (1)" width="419" height="338" border="0" /></a>

After that, just use the defaults for the rest of the settings. Make note of the Deployment ID. You are going to need it later. Mine was PS120003.

You now have a task sequence that is set to be available in 19 years, how does that help me? The client will download the policy for this deployment (you can speed it up with machine policy updates) now and set the active time to the year 2033. We can use a script to start the task sequence, and we can deploy this script as an application to the application catalog. Here’s how you can set up the application. First off, make sure you have the client settings Powershell execution policy to bypass:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image2-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image2 (2)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image2-2_thumb.png" alt="image2 (2)" width="444" height="242" border="0" /></a>

Here is the script you will want to use. Save this as a new Powershell script wherever you store content. Edit the first line to be $DeploymentID = “your deployment ID”

 
<pre class="lang:ps decode:true " >$DeploymentID = "PS120003"
$strQuery = "Select * from CCM_Scheduler_ScheduledMessage where ScheduledMessageID like '" + $DeploymentID + "%'"
Get-WmiObject -Query $strQuery -Namespace root\ccm\policy\machine\actualconfig | ForEach-Object {
     $ActiveTime = $_.ActiveTime
     $ActiveTime = "2012" + $ActiveTime.Substring(4)
     $_.ActiveTime = $ActiveTime
     $_.Put()
     $ScheduleID = $_.ScheduledMessageID
}
$strQuery = "Select * from CCM_SoftwareDistribution where ADV_AdvertisementID='" + $DeploymentID + "'"
Get-WmiObject -Namespace "root\CCM\Policy\Machine\ActualConfig" -Query $strQuery | ForEach-Object {
     $_.ADV_MandatoryAssignments = "True"
     $_.Adv_ActiveTime = $ActiveTime
     $_.ADV_RepeatRunBehavior = "RerunAlways"
     $_.Put()
}
$WMIPath = "\\.\root\ccm:SMS_Client"
$SMSwmi = [wmiclass] $WMIPath
[Void]$SMSwmi.TriggerSchedule($ScheduleID)
"$Error" &gt; "c:\TS_ErrorLog.txt"
"Done" &gt;&gt; "C:\TS_ErrorLog.txt"</pre> 


Now create an application like you would if you were creating a script application.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image3-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image3 (2)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image3-2_thumb.png" alt="image3 (2)" width="490" height="405" border="0" /></a>

Set the application catalog settings however you want them. I just left the defaults and clicked next. Then, add a new deployment type:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image4-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image4 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image4-1_thumb.png" alt="image4 (1)" width="407" height="211" border="0" /></a>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image5-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image5 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image5-1_thumb.png" alt="image5 (1)" width="423" height="208" border="0" /></a>

Make sure the content location is wherever you stored the script and the installation program command line should be: Powershell.exe –ExecutionPolicy Bypass –File “filename.ps1”

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-3-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image (3)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-3_thumb.png" alt="image (3)" width="451" height="388" border="0" /></a>

Hit next and add a File System detection rule. The script will log any errors to a file called “TS_ErrorLog.txt” on the root of C. We will set up the detection method to find that file so the application doesn’t error out once it is finished running.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image1-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image1 (2)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image1-2_thumb.png" alt="image1 (2)" width="460" height="456" border="0" /></a>

Click Ok, and then next. Here you need to make it install for system, and logon requirement can be only when a user is logged on, or whether or not a user is logged on:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image8-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image8 (2)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image8-2_thumb.png" alt="image8 (2)" width="443" height="386" border="0" /></a>

Hit next and now look at the requirements. Since this application is going to be deploying a task sequence, you might want to add a requirement so it can only run on Windows XP – 8.1 and skip server OS:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image9-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image9 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image9-1_thumb.png" alt="image9 (1)" width="350" height="376" border="0" /></a>

Hit Next, Next, Next, Close (to finish creating the deployment type) and then Next –&gt; Next to finish creating the Application. Now, deploy this as you would any other application catalog application, as available to the user collection you want it to show up in. You can check “Require administrator approval” if you want, depending on who you are deploying this to it might be a good idea. Finish the wizard using all the defaults for scheduling, user experience and alerts. And you’re finished! Oh, maybe we should test this.

Go to a test machine that has the task sequence deployed to it from the first step (as available sometime in the future). Open up software center and you will see there is no task sequence listed:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image10-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image10 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image10-1_thumb.png" alt="image10 (1)" width="431" height="180" border="0" /></a>

Open up the application catalog, and you will see the application we just deployed:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image11-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image11 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image11-1_thumb.png" alt="image11 (1)" width="458" height="254" border="0" /></a>

Click Install, and then Yes:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image12-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image12 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image12-1_thumb.png" alt="image12 (1)" width="244" height="153" border="0" /></a>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image13-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image13 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image13-1_thumb.png" alt="image13 (1)" width="244" height="150" border="0" /></a>

The task sequence should now start!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image15-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image15 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image15-1_thumb.png" alt="image15 (1)" width="361" height="166" border="0" /></a>

And now you’re done!