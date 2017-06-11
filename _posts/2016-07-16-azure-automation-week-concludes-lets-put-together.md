---
id: 621
title: 'Azure Automation Week Concludes! Let&rsquo;s put it all together!'
date: 2016-07-16T02:37:06+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=621
permalink: /azure-automation-week-concludes-lets-put-together/
categories:
  - Azure Automation
  - ConfigMgr
tags:
  - Azure Automation
  - ConfigMgr
  - OSD
  - Task Sequence
---
Whew, this week flew by! We learned how to <a href="http://www.ephingadmin.com/azure-automation-week/" target="_blank">set up Azure Automation</a>, how to <a href="http://www.ephingadmin.com/azure-automation-week-run-runbook-without-cmdlets/" target="_blank">run a runbook with the REST API</a>, how to do a <a href="http://www.ephingadmin.com/azure-automation-week-advanced-offline-domain-join-azure-automation/" target="_blank">more advanced offline domain join</a>, and how to <a href="http://www.ephingadmin.com/azure-automation-week-get-input-scsm-portal/" target="_blank">get input from the SCSM portal</a>. Today, we are going to put all that we learned into a task sequence!

First off, we just need a task sequence that installs Windows 10 Enterprise. I used the default task sequence but am joining the computer to a workgroup. We want to use our fancy new Azure Automation runbook to add the computer to the domain! Here’s my apply network settings step:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-57.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-57.png" alt="image" width="496" height="484" border="0" /></a>

The only other change I did was to add an Azure Automation script at the end. I’m using the <a href="http://www.ephingadmin.com/azure-automation-week-run-runbook-without-cmdlets/" target="_blank">function from Tuesday</a> and the runbooks from <a href="http://www.ephingadmin.com/azure-automation-week-advanced-offline-domain-join-azure-automation/" target="_blank">Wednesday</a> and <a href="http://www.ephingadmin.com/azure-automation-week-get-input-scsm-portal/" target="_blank">Thursday</a>. I did change my domain join runbook from Wednesday. Instead of just returning the domain join file, I’m also returning the domain join certificate. I exported the certificate as a .reg file and put the text in the script:

 
<pre class="lang:ps decode:true " >$CertString = @'
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SystemCertificates\MY\Certificates\EA767B6091B141417C9A9BE93E5026AEE8D050AE]
BLAHBLAHLBAH
'@

$returnObject = @{
    'Cert'=$CertString
    'OfflineDomainJoin'="$FileContent"
}
return (ConvertTo-Json $returnObject)</pre> 


Now I have all I need to domain join a computer from anywhere and put it on the network through direct access! My task sequence ends in a script that calls these runbooks:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-58.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-58.png" alt="image" width="626" height="484" border="0" /></a>

The script is my Run-AzureRunbook with this code:

 
<pre class="lang:ps decode:true " >$SecretParams = @{
    'UserName'= ''
    'Password'=''
    'AutomationAccount' = ''
    'adTenant' = ''
}

$Results = Run-AzureRunbook -RunBookName 'SCSMPortalQuestionsAnswers' -HybridWorkerGroup 'OSD_Automation_Group' -Params '"TicketID":"SR18"' @SecretParams
$OffDomainJoinResults = Run-AzureRunbook -RunBookName 'OfflineDomainJoin' -HybridWorkerGroup 'OSD_Automation_Group' -Params "`"ComputerName`":`"$($Results.'Computer Name')`"" @SecretParams
Rename-Computer -NewName $Results.'Computer Name' -Force
$OffDomainJoinResults.Cert &gt; 'c:\cert.reg' 
regedit /s c:\cert.reg
Remove-Item -Path 'c:\cert.reg' -Force
$OffDomainJoinResults.OfflineDomainJoin &gt; 'c:\djoin.txt'
djoin.exe /requestodj /loadfile c:\djoin.txt /windowspath C:\Windows /localos
Remove-Item -Path 'c:\djoin.txt' -Force</pre> 


As you can see, I’m first getting the information from the ticket, then running the domain join runbook. I then rename the computer based on the ticket input, import the certificate for direct access, and do the domain join! Now you just need to restart after and you have a computer on the domain!

Beginning:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-59.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-59.png" alt="image" width="369" height="484" border="0" /></a>

Middle:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-60.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-60.png" alt="image" width="564" height="484" border="0" /></a>

End:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-61.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-61.png" alt="image" width="368" height="484" border="0" /></a>

That’s it for Azure Automation week! I hope you all had fun and learned something!