---
id: 108
title: Deploy a Task Sequence to All Systems and Keep Your Job!
date: 2014-12-02T17:47:13+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=108
permalink: /deploy-a-task-sequence-to-all-systems-and-keep-your-job/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Deployments
  - SCCM
  - Task Sequence
---
Disclaimer: You’re going to read about a way to save yourself from reimaging your environment if you deploy a task sequence to the wrong collection. DO NOT TEST IN PRODUCTION.

One of the shortcomings of ConfigMgr is how easy it is to reimage your environment. With one wrong click, you could find yourself updating your resume on a personal computer because all the work ones are reimaging… For my MMS session with Trevor Sullivan on Powershell events, I wrote a script to save yourself from deploying task sequences to the wrong collection  using WMI events.

The script uses <a href="https://powerevents.codeplex.com/" target="_blank">PowerEvents</a> to set up the WMI event, so be sure to grab and copy the files to your favorite <a href="http://msdn.microsoft.com/en-us/library/dd878350%28v=vs.85%29.aspx" target="_blank">Powershell Module</a> path. If you haven’t worked with WMI events before, here is a quick run down. We need a filter (what am I looking for?), a consumer (what do I do?), and a binding which binds the filter and consumer together (makes the consumer run whenever the filter is triggered).

So, here is the script:
<pre class="lang:ps decode:true">$Query = "Select * from __InstanceCreationEvent Within 30 Where TargetInstance ISA 'SMS_DeploymentSummary' AND TargetInstance.FeatureType='7' AND TargetInstance.DeploymentIntent='1'"
$Filter = New-WmiEventFilter -Name RequiredDeploymentFilter1 -Query $Query -EventNamespace 'root\sms\site_ps1'
$Consumer = New-WmiEventConsumer -Name RequiredDeploymentConsumer1 -Namespace 'root\sms\site_ps1' -ConsumerType CommandLine -CommandLineTemplate "Powershell.exe -File C:\OhNo.ps1 -DeploymentID %TargetInstance.DeploymentID%"
New-WmiFilterToConsumerBinding -Filter $Filter -Consumer $Consumer</pre>
What does this script do? The first line is the WMI event query. If you are new to WMI event queries, here is how it works:

“Select * from __InstanceCreationEvent” –&gt; Whenever a new WMI instance (so new Deployment) is made, this event is triggered. We want to look for all new deployments, so that’s why I’m looking for creation events.

“Within 30” –&gt; This tells WMI how often it should check for the event. If you make it check too often, you may have performance issues, so play around with this number for your environment. As is, it will check every 30 seconds

“Where TargetInstance ISA ‘SMS_DeploymentSummary’” –&gt; There are multiple WMI class instances that are created when a new deployment is created. I just chose this one.

“AND TargetInstance.FeatureType=’7’” –&gt; FeatureType is a property of DeploymentSummary. A FeatureType of 7 means Task Sequence

“AND TargetInstance.DeploymentIntent=’1’” –&gt; DeploymentIntent is also a property of DeploymentSummary. DeploymentIntent of 1 means required.

This query looks for new required task sequence deployments every 30 seconds. Now that we have the query, we can create the Filter, which is line two of the script. All filters need a unique name to identify them, a query, and the namespace of the WMI event. Line two sets all that up and stores the filter into the variable $Filter.

After the filter, we need a consumer. Since I tend to love Powershell, I wrote a consumer that launches a Powershell script. Here is the breakdown of the command:

New-WMIEventConsumer –Name RequiredDeploymentConsumer1 –&gt; This is a unique name for the consumer

-Namespace ‘root\sms\site_ps1’ –&gt; This is the Namespace of the WMI filter

-ConsumerType CommandLine –&gt; I want to launch a Powershell script, so the consumer type will be command line.

-CommandLineTemplate “Powershell.exe –File C:\OhNo.ps1 –DeploymentID %TargetInstance.DeploymentID%” –&gt; This sets up the command line that will be run. TargetInstnace is the instance that triggered the command (So the new SMS_DeploymentSummary instance) and .DeploymentID will be the deploymentID. So when it is run, the command line that will run is:
Powershell.exe –File C:\OhNo.ps1 –DeploymentID PS100005
That will be the command run if the new deployment ID is PS100005.

The file C:\OhNo.ps1 is the Powershell script I wrote to do the rest of the work.

Now that we have the Filter and Consumer finished, we need something to bind them together. That’s what the last line does. It runs the command New-WMIFilterToConsumerBinding and binds the two.

That’s it! Well, sort of. Now, ever time you make a new deployment the script C:\OhNo.ps1 will be run. What is in that script?
<pre class="lang:ps decode:true">[CmdletBinding()]
param(
    [string] $DeploymentID
)
 
$Query = "Select * from SMS_DeploymentSummary "`
     + "join SMS_Collection on SMS_DeploymentSummary.CollectionID = SMS_Collection.CollectionID"`
     + " where SMS_DeploymentSummary.DeploymentID = '$DeploymentID'"
 
$Collection = Get-WmiObject -Namespace 'root\sms\site_ps1' -Query $Query
 
If ($Collection.SMS_Collection.MemberCount -gt 5) {
 
    $Advertisement = Get-WmiObject -Namespace 'root\sms\site_ps1' -Class SMS_Advertisement -Filter "AdvertisementID='$DeploymentID'"
    $Advertisement.Get()
    $Advertisement.PresentTime = "20321225111300.000000+***"
    $Advertisement.ExpirationTime = "20331226111300.000000+***"
    $Advertisement.put()
 
    $strBody = "Hey, I just saved your job!`nDelayed required deployment of task sequence " + $Collection.SMS_DeploymentSummary.SoftwareName`
     + " to collection " + $Collection.SMS_Collection.Name
    
    Send-MailMessage -Body $strBody -From "OhNoScript@home.local" -SmtpServer localhost -Subject "Delayed Deployment" -To "NotifyMe@Home.Local"
}</pre>
Now, what does this script do? It takes the DeploymentID fed to it by the WMI Event and looks to see how many computers are in the collection it is deployed to. Then, it changes the Deployment Deadline to Christmas, 2033 if the number of computers deployed to is over 5. Lastly, it sends a e-mail message notifying me that the deployment deadline was changed and that it saved my job!

And that’s it! If you want to implement this in your environment, you’ll need to change the namespaces to your correct namespace (if your site code isn’t PS1 – in both scripts), the number of seconds it waits before triggering this (in the WMI filter query), and the member count of the collection (you’ll probably want to be able to deploy to collections larger 5 members). The script is set up to send an e-mail to the SmtpServer localhost. It is set up this way so I can do testing with <a href="https://papercut.codeplex.com/" target="_blank">Papercut</a>. Change the information to work in your environment, or change what it does when it changes the deployment (maybe write to a text file?)

I purposefully didn’t make this too easy to implement in your environment to force you guys to understand and test it out first! Let me know if you want any more information on this in the comments.

Lastly, this only works for NEW deployments. Anyone can go in and reset the deploy time to now and the deployment will start. This is only meant to prevent the creation of a deployment to the wrong collection.