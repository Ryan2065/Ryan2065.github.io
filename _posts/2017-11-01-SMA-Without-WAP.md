---
id: 711
title: 'SMA Without the WAP'
date: 2017-11-01T01:10:39+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=711
permalink: /SMA-Without-WAP/
categories:
  - PowerShell
  - SMA
---

# SMA Without the WAP

When I first started looking at SMA, I pulled together a list of the 
requirements based on install guides. One of the requirements I listed was the 
Windows Azure Pack. Why? It's listed in all of the install guides:

1) [You can install the web service on any machine that can communicate with Windows Azure Pack and an instance of SQL Server.](https://docs.microsoft.com/en-us/system-center/sma/deploy)
2) [Click Add next to Windows Azure Pack: Portal and API Express, and then click Install](http://blogs.catapultsystems.com/mdowst/archive/2013/10/21/service-management-automation-lab-step-by-step-install/)
3. [When the installation is complete, you can add the SMA installation to Windows Azure Pack ](http://windowsitpro.com/orchestrator/install-orchestrator-sma)

I then installed SMA and had problems connecting to the Windows Azure Pack. I 
ran a few commands and soon realized SMA was functioning correctly without the 
WAP. After a few days and weeks of testing, I came to the conclusion that SMA 
does not need WAP. Everything works without it! 

What features does the WAP give, and how can you live without it? 

### Import Runbooks & Modules
WAP allows you to easily import a runbook. How can you live without this? Easy -
 just use the PowerShell cmdlets!

```
Import-SMARunbook -path "C:\PathToPS1.ps1" -WebServiceEndpoint 'https://MyEndpoint'
```

You can also import modules with the WAP. Again, another easy task with PowerShell. Simply zip up your module and run this command:

```
Import-SMAModule -Path "C:\PathToZip.zip" -WebServiceEndpoint 'https://MyEndpoint'
```

### View Job Information
WAP allows you to view what jobs are running in your environment, and some 
general information about them. Again, we can overcome this with the PowerShell 
cmdlets!

```
#Get all jobs
Get-SMAJob -WebServiceEndpoint 'https://MyEndpoint'

#Get jobs for a specific runbook
Get-SMAJob -RunbookName 'My Runbook' -WebServiceEndpoint 'https://MyEndpoint'
```

But Ryan, the WAP shows me in a nice UI this information, so it's much easier to 
parse! Guess what, PowerShell can do that also!


```
#Get all jobs
Get-SMAJob -WebServiceEndpoint 'https://MyEndpoint' | Out-GridView

#Get jobs for a specific runbook
Get-SMAJob -RunbookName 'My Runbook' -WebServiceEndpoint 'https://MyEndpoint' | Out-GridView
```

### View Job Streams
Lastly, one other thing the WAP will be used for is to view the logs of the job, also known as the job streams. If you've been following along, you probably know what I'm going to say next. PowerShell:

```
$SMAJob = Get-SMAJob -RunbookName 'My Runbook'
Get-SMAJobOutput -id $SMAJob[0].JobId -WebServiceEndpoint 'https://MyEndpoint' | Out-GridView
```

### Other Actions
There are a few other little actions here and there the WAP lets you do. For instance scheduling a runbook (Set-SMASchedule), starting a runbook (Start-SMARunbook), and configuring a runbook (Set-SMARunbookConfiguration). 

As you can see, we don't need the WAP! All of this can easily be done in 
PowerShell. Will the WAP make it a little easier? Sure, but then you have the 
overhead of installing and maintaining Windows Azure Pack to simply read log 
files. If Windows Azure Pack already exists in your environment for other 
technologies, adding a SMA environment onto it is a no-brainer. However, if you 
are planning on using SMA and don't already have the WAP, consider avoiding it 
completely and going straight PowerShell!

That's it for today! Check back later to see some SMA cmdlets I've been working 
on to make managing and devloping in SMA even easier!