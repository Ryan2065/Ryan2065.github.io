---
id: 608
title: 'Azure Automation Week&ndash;Get Input from the SCSM Portal'
date: 2016-07-14T22:09:37+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=608
permalink: /azure-automation-week-get-input-scsm-portal/
categories:
  - Azure Automation
  - SCSM
tags:
  - Azure Automation
  - Powershell
  - SCSM
---
<span style="font-size: small;">Most of the time when we are imaging computers, there is a reason for it. This reason will be tracked in some form of ticketing system, so why are we asking questions in the task sequence? We have all the answers already! In this post, I’m going to show you how to get the answers to portal questions in SCSM, and then return those as JSON!</span>

<span style="font-size: small;">In order to do this, there are some hurdles we must overcome. They are:</span>

<span style="font-size: small;">1) HybridRunbookWorkers run under the System context
2) String is returned, not an object
3) SCSM</span>

<span style="font-size: small;">One of the newer features of Azure Automation is the ability to change the context your hybrid worker runs under. Up until a few months ago we could only run scripts under the System context. Now, we have the ability to run under any user context by changing one simple setting:</span>

<span style="font-size: small;">Go to portal.azure.com and open up your Azure Automation account.</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-48.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-48.png" alt="image" width="644" height="420" border="0" /></a>

Select Hybrid Worker Groups, then select your group name:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-49.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-49.png" alt="image" width="644" height="203" border="0" /></a>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-50.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-50.png" alt="image" width="644" height="247" border="0" /></a>

Notice how it says “Run As –“? This is telling us the run as account is the default. On the right side, click Hybrid worker group settings, and change it from Default to Custom:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-51.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-51.png" alt="image" width="644" height="225" border="0" /></a>

Be sure to only give the hybrid worker the least amount of privileges it needs! In my case, I’ll select Domain Administrator…
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-52.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-52.png" alt="image" width="244" height="187" border="0" /></a></p>
Click Save and you’ll see Run As change:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-53.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-53.png" alt="image" width="644" height="116" border="0" /></a>

Now that we have our hybrid worker running under the correct credentials, we can continue onto our script! I created a new PowerShell runbook called SCSMPortalQuestionsAnswers for this.

First, I need to ask for the ticket ID, get the name of the SCSM server, and open a PSSession to the SCSM Server:

 
<pre class="lang:ps decode:true " >param (
    $TicketID
)

$SCSMServer = Get-AutomationVariable -Name "SCSMServer"

$Session = New-PSSession -ComputerName $SCSMServer</pre> 


Now I need to get all the questions and answers from the SCSM portal. I’ll use the SMELTS to get this:

 
<pre class="lang:ps decode:true " >$GetResultsScriptBlock = {
    $TicketID = $args[0]
    Import-Module 'C:\Program Files\Common Files\SMLets\SMLets.psd1'
    $RequestClass = Get-SCSMClass -Name System.WorkItem.ServiceRequest$ 
    $WorkItem = Get-SCSMObject -Class $RequestClass -Filter "ID -eq $TicketID"
    [xml]$UserInput = $WorkItem.UserInput
    $QuestionAnswers = @{}
    $UserInput.UserInputs.UserInput | ForEach-Object {
        $QuestionAnswers[$_.Question] = $_.Answer
    }
    return (ConvertTo-Json $QuestionAnswers)
}</pre> 


If you notice, I’m converting the object I’m creating to json. If you ever want to send an object back through the REST API, this is how you have to do it.

Lastly, I need to call the scriptblock and return the string!

 
<pre class="lang:ps decode:true " >return (Invoke-Command -Session $Session -ScriptBlock $GetResultsScriptBlock -ArgumentList $TicketID)</pre> 


Now, I simply need to create a request offering for imaging computers in SCSM and publish it to the portal.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-54.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-54.png" alt="image" width="171" height="244" border="0" /></a>

Once I answer those questions, I can take the ticket ID and send it to my runbook:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-55.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-55.png" alt="image" width="644" height="23" border="0" /></a>

The results are exactly what I expect, an object with the questions and answers:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-56.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-56.png" alt="image" width="644" height="86" border="0" /></a>

And there you have it! I can now either take the HTA out of my task sequence or greatly reduce the number of questions I ask in my task sequence. For instance, I could simply ask for the ticket ID and get all the questions answered! Or I could ask for the MAC address in the ticket and make this a zero touch!