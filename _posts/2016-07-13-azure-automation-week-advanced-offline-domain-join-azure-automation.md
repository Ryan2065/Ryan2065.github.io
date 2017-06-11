---
id: 587
title: 'Azure Automation Week: Advanced Offline Domain Join With Azure Automation!'
date: 2016-07-13T21:48:56+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=587
permalink: /azure-automation-week-advanced-offline-domain-join-azure-automation/
categories:
  - Azure Automation
  - ConfigMgr
tags:
  - Azure Automation
  - Offline Domain Join
---
<span style="font-size: small;">Azure automation week continues! Today we are going to look at joining computers to a domain through Azure Automation. I always liked the idea of offline domain join, but never really used it since I don’t prestage computers. I then got the idea, what if I could join a computer to the domain from offsite without having to prestage it first? </span>

<span style="font-size: small;">To do this, we will need to set some things up! First off, if you want the domain joined computer to get group policies right away, you’ll need direct access. If you are on a lab, go ahead and set it up with <a href="https://blogs.technet.microsoft.com/askpfeplat/2013/08/18/how-to-setup-your-own-direct-access-lab-with-windows-server-2012/" target="_blank">these instructions.</a> Don’t worry if you don’t have direct access. You can still do this demo, it just won’t be as cool.</span>

<span style="font-size: small;">Second, I’ll be using the PowerShell ISE to work on the Azure Automation scripts. You will need the <a href="https://azure.microsoft.com/en-us/blog/announcing-azure-automation-powershell-ise-add-on/" target="_blank">Azure Automation PowerShell ISE add-on</a> to follow along. This really is a must have add-on if you want to develop in Azure Automation! </span>

<span style="font-size: small;">Third, I need the Active Directory cmdlets installed on my hybrid worker computer. My offline domain join script will use those cmdlets to create the computer account, so they are needed on the server the script runs on.</span>

<span style="font-size: small;">Once you have the add-on installed, open your ISE. You should see it on the right side when you open the console:</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-35.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-35.png" alt="image" width="327" height="484" border="0" /></a>

Be sure to sign into your account, then make sure the right subscription and automation account are selected.

To create a computer account, I’m going to need the credentials of a user that has those permissions. In Azure Automation, we can store credential objects in the cloud, and access them through the Azure cmdlets in our scripts. We can use a credential asset for this. Go to the Assets tab, then select new:
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-36.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-36.png" alt="image" width="268" height="484" border="0" /></a></p>
<p align="left">Choose the asset type and give it a name:</p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-37.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-37.png" alt="image" width="244" height="142" border="0" /></a></p>
<p align="left">Once you click Ok, it will ask for the username and password. You must specify the username in Domain\UserName format:</p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-38.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-38.png" alt="image" width="244" height="129" border="0" /></a></p>
<p align="left">Click Ok, then you have your new credential object! Right now, it is only local. You must hit “Upload” on the bottom to send it to the cloud. Once you do, the status will change to “In Sync”</p>
<p align="left"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-39.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-39.png" alt="image" width="644" height="163" border="0" /></a></p>
Follow this procedure to create another Asset of type Variable for the Direct Access server name, if you have one:
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-40.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-40.png" alt="image" width="244" height="149" border="0" /></a></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-41.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-41.png" alt="image" width="244" height="173" border="0" /></a></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-42.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-42.png" alt="image" width="244" height="93" border="0" /></a></p>
Make sure you upload this to the cloud!

Now, you need to create a PowerShell runbook. Go to the Runbooks tab, and select Create New at the bottom. Enter a name for it (Mine is OfflineDomainJoin) and make sure it is a Script type. After it’s created, the script should be open in the window. First off, we need to know some parameters. To make this easier, we will only ask for ComputerName and Path. This way we can create a computer in the right location:
<pre class="lang:ps decode:true ">param (
    $ComputerName, 
    $Path = 'CN=Computers,DC=Home,DC=Lab'
)</pre>
I’ll also need to import the active directory module, so be sure to say Import-Module ActiveDirectory

In the ISE, we are now going to access the DirectAccessServer variable. Type in $DAServer = then, hit insert on the right side:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-43.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-43.png" alt="image" width="134" height="244" border="0" /></a>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-44.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-44.png" alt="image" width="244" height="69" border="0" /></a>

You’ll want to do the same for your AD credential object.

The rest of the script is pretty straight forward. You’re going to create the AD computer, put it in the right OU, then add it to the Direct Access AD group (assuming you have DA):
<pre class="lang:ps decode:true ">$dName = "CN=$($Computername),$($Path)"
$DirectAccessADGroup = 'Direct Access Clients'
$null = New-ADComputer -Name $ComputerName -SAMAccountName $ComputerName -Path $Path -Enabled $true -Credential $ADCreds -PassThru
$null = Add-ADGroupMember $DirectAccessADGroup -Members $dName -Credential $ADCreds</pre>
Now, for the domain join piece. I could not for the life of me get this to work on the same server as the hybrid runbook worker. The hybrid runbook worker will run your scripts as the system context, so you have to run it as a domain user to run djoin. The only thing I could get working was starting a PowerShell process on a remote machine to do the djoin step. Because of this, make sure DirectAccessServer is a remote computer!

The djoin step involves running a scriptblock on a remote machine. This scriptblock does the djoin step and then copies the file from the DiretAccessServer to the hybrid runbook worker:
<pre class="lang:ps decode:true ">$Session = New-PSSession -ComputerName $DAServer -Credential $ADCreds
$djoinScriptBlock = {
    $ComputerName = $args[0]
    $HybridWorkerServer = $args[1]
    djoin.exe /provision /machine "$ComputerName" /domain "Home.Lab" /policynames "DirectAccess Client Settings" /savefile "c:\$ComputerName.txt" /reuse
    Copy-Item "c:\$ComputerName.txt" "\\$HybridWorkerServer\c$\" -Force
    Remove-Item "c:\$ComputerName.txt" -Force
}
$djoinResults = Invoke-Command -Session $Session -ScriptBlock $djoinScriptBlock -ArgumentList $ComputerName,$env:COMPUTERNAME
</pre>
Lastly, I need to get the contents of the file, then remove the file, then return the file contents:
<pre class="lang:ps decode:true ">if (Test-Path "C:\$ComputerName.txt") {
    $FileContent = Get-Content "C:\$ComputerName.txt"
    Remove-Item "C:\$ComputerName.txt" -Force
    $count=100
}
return $FileContent</pre>
Now, hit the “Upload Draft” button, then “Publish Draft” and now we are ready to test it out!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-45.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-45.png" alt="image" width="244" height="42" border="0" /></a>

I’ll use my function from yesterday to invoke the runbook through PowerShell:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-46.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-46.png" alt="image" width="1028" height="35" border="0" /></a>

After a minute, I should get results:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/07/image-47.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/07/image_thumb-47.png" alt="image" width="644" height="86" border="0" /></a>

I can now output $Results to a file and do an offline domain join with it! If that computer also has your direct access certificate, you will have direct access and start receiving group policies on the computer!

That’s it for today! Join me tomorrow when I build off the information we learned today to pull data from SCSM into Azure Automation. Then on Friday I’ll shove all of this into a SCCM task sequence!

Full script here:

 
<pre class="lang:ps decode:true " >param (
    $ComputerName, 
    $Path = 'CN=Computers,DC=Home,DC=Lab'
)

Import-Module ActiveDirectory

$DAServer = Get-AutomationVariable -Name "DirectAccessServer"
$ADCreds = Get-AutomationPSCredential -Name "ADCredentials"

$dName = "CN=$($Computername),$($Path)"
$DirectAccessADGroup = 'Direct Access Clients'
$null = New-ADComputer -Name $ComputerName -SAMAccountName $ComputerName -Path $Path -Enabled $true -Credential $ADCreds -PassThru
$null = Add-ADGroupMember $DirectAccessADGroup -Members $dName -Credential $ADCreds

$Session = New-PSSession -ComputerName $DAServer -Credential $ADCreds
$djoinScriptBlock = {
    $ComputerName = $args[0]
    $HybridWorkerServer = $args[1]
    djoin.exe /provision /machine "$ComputerName" /domain "Home.Lab" /policynames "DirectAccess Client Settings" /savefile "c:\$ComputerName.txt" /reuse
    Copy-Item "c:\$ComputerName.txt" "\\$HybridWorkerServer\c$\" -Force
    Remove-Item "c:\$ComputerName.txt" -Force
}
$djoinResults = Invoke-Command -Session $Session -ScriptBlock $djoinScriptBlock -ArgumentList $ComputerName,$env:COMPUTERNAME

if (Test-Path "C:\$ComputerName.txt") {
    $FileContent = Get-Content "C:\$ComputerName.txt"
    Remove-Item "C:\$ComputerName.txt" -Force
    $count=100
}
return $FileContent</pre> 
