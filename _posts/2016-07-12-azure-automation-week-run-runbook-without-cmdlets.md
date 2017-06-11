---
id: 558
title: 'Azure Automation Week: Run a Runbook Without the Cmdlets!'
date: 2016-07-12T21:56:53+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=558
permalink: /azure-automation-week-run-runbook-without-cmdlets/
categories:
  - Azure Automation
  - ConfigMgr
tags:
  - Azure Automation
  - REST API
---
<span style="font-size: small;">Now that I have an Azure Automation account, and a user with the correct permissions, I have the ability to run Azure Automation runbooks without relying on the cmdlets or webhooks. I don’t want to have to rely on cmdlets in WinPE, and I also don’t want to have to rely on webhooks because you cannot get the results back from them. </span>

<span style="font-size: small;">Before I can start, I need a runbook to test! So, let’s fire up portal.azure.com and select our Automation Account:</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-26.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border-width: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-26.png" alt="image" width="644" height="275" border="0" /></a>

Now, you’ll want to select Runbooks:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-27.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-27.png" alt="image" width="644" height="442" border="0" /></a>

Add a runbook

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-28.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-28.png" alt="image" width="644" height="249" border="0" /></a>

Quick Create

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-29.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-29.png" alt="image" width="540" height="484" border="0" /></a>

Then give it a name and use the type PowerShell:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-30.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-30.png" alt="image" width="341" height="484" border="0" /></a>

Now you have a screen that is similar to the PowerShell ISE window. I’m not going to do anything fancy here. I’ll just have a parameter and return the parameter:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-31.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-31.png" alt="image" width="644" height="308" border="0" /></a>

Save it, then publish it:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-32.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-32.png" alt="image" width="644" height="211" border="0" /></a>

You now have a runbook you can use in a script!

Now, we want to be able to call the runbook in PowerShell and wait for a response. To do this, I’ll be using the <a href="https://msdn.microsoft.com/en-us/library/azure/mt163781.aspx" target="_blank">Azure Automation REST API</a>. There is a great guide on <a href="http://www.laurierhodes.info/?q=node/118" target="_blank">Laurie Rhode’s blog that details how to do this in PowerShell</a>, but they don’t leave us with a workable function. I took this information and put it into a function so we can easily use it in OSD! Here’s my function:

 
<pre class="lang:ps decode:true " >Function Run-AzureRunbook {
    Param (
        $AutomationAccount, 
        $adTenant, 
        $UserName, 
        $Password,
        $RunBookName,
        $RunBookParamHash,
        $HybridWorkerGroup,
        $Params
    )
    $ARMResource = 'https://management.core.windows.net/'
    $resourceAppIdURI = 'https://management.core.windows.net/'
    $ClientID = '1950a258-227b-4e31-a9cf-717495945fc2'
    $FullUserName = "$UserName@$($adTenant)"
    $TokenEndpoint = "https://login.windows.net/$($adTenant)/oauth2/token"

    $UserAuthPayload = "resource=$($resourceAppIdURI)&amp;client_id=$($clientId)"+"&amp;grant_type=password&amp;username=$($FullUserName)&amp;password=$($password)&amp;scope=openid"
    $null = [System.Reflection.Assembly]::LoadWithPartialName("System.Web")
 
$payload =@"
   $($UserAuthPayload),
   $([System.Web.HttpUtility]::UrlEncode($ARMResource)),
   $([System.Web.HttpUtility]::UrlEncode($FullUserName)),
   $([System.Web.HttpUtility]::UrlEncode($Password)),
"@

    $Header = @{
     "Content-Type" = "application/x-www-form-urlencoded";
    }

    try {
        $AuthResult = Invoke-RestMethod -Uri $TokenEndpoint -Method Post -body $Payload -Headers $Header
    }
    catch {
        $ThrowMessage = "Could not create the authorization token. Error Message: $($_.Exception.Message)"
        throw $ThrowMessage
        Exit
    }

    $JobId = [GUID]::NewGuid().ToString()
    $RequestHeader = @{
        "Content-Type" = "application/json";
        "x-ms-version" = "2013-08-01";
        "Authorization" = "$($authResult.token_type) $($authResult.access_token)"
    }
    $Body = @"
        {
           "properties":{
           "runbook":{
               "name":"$RunBookName"
           },
           "parameters":{
                $Params
           },
           "runOn":"$HybridWorkerGroup",
           "isEncrypted":1
          }
       }
"@
    $URI = 'https://management.azure.com' + "$($AutomationAccount)/jobs/$($jobid)?api-version=2015-10-31"
    try {
        $Response = Invoke-RestMethod -Uri $URI -Method Put -body $body -Headers $RequestHeader
    }
    catch {
        $ThrowMessage = "Could not start job. Error Message: $($_.Exception.Message)"
        Throw $ThrowMessage
        Exit
    }
    $LoopStart = Get-Date
    $Loop = $true
    while($Loop) {
        $job = $null
        try {
            $job = Invoke-RestMethod -Uri $URI -Method GET -Headers $RequestHeader
        }
        catch {
            $ThrowMessage = "Could not get the job results. Error Message: $($_.Exception.Message)"
            Throw $ThrowMessage
            exit
        }
        $Script:Status = $Job.Properties.provisioningState
        $Loop = (($Status -ne 'Succeeded') -and ($Status -ne 'Failed') -and ($Status -ne 'Suspended') -and ($Status -ne 'Stopped'))
        $TimeElapsed = (Get-Date) - $LoopStart
        if ($TimeElapsed.Minutes -ge 5) { $Loop = $false }
    }
    $responseURI = 'https://management.azure.com' + "$($AutomationAccount)/jobs/$($jobid)/output?api-version=2015-10-31"
    $Response = $null
    try {
        $Response = Invoke-RestMethod -Uri $responseURI -Method Get -Headers $RequestHeader
    }
    catch {
        $ThrowMessage = "Could not get the job output. Error Message: $($_.Exception.Message)"
        Throw $ThrowMessage
        exit
    }
    return $Response
}
</pre> 


The parameters and where you get them from are:

RunbookName: The name of the runbook you created
Params: The parameters in this format:  “myinput”:”datahere”
HybridWorkerGroup: The name of the hybrid worker group from  yesterday
UserName: OSD Service Account username that was set up yesterday
Password: OSD Service Account password that was set up yesterday
Automation Account: If you go to your runbook and then select settings –&gt; Properties, you’ll see the Automation Account. It’s a VERY long string that starts with /subscriptions
adTenant: The adTenant of UserName. If you don’t already know it, you can go to manage.windowsazure.com, go down to Active Directory, select your domain, find the username, then it’s whatever is after @.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-33.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-33.png" alt="image" width="644" height="106" border="0" /></a>

&nbsp;

Now, run the runbook with this command, and you should get the right data returned!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-34.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-34.png" alt="image" width="644" height="38" border="0" /></a>