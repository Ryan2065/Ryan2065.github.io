---
id: 60
title: Using the PSCmdlets in the Now Micro Right Click Tools
date: 2015-04-02T23:03:43+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=60
permalink: /using-the-pscmdlets-in-the-now-micro-right-click-tools/
categories:
  - ConfigMgr
  - Powershell
  - Right Click Tools
tags:
  - Cmdlets
  - ConfigMgr 2012
  - Powershell
---
I wanted to take some time to show you all how to use the cmdlets that come with the <a href="http://www.nowmicro.com/recast/" target="_blank">Now Micro Right Click Tools</a>. The cmdlets were made to be simple and easy to use, but one of them might not be that straightforward. The Powershell Cmdlets were made to quickly run scriptblocks on lists of computers. Here are the available commands:

Get-CollectionMembers: Returns an array of computer names from a Collection ID or Collection Name
Start-ClientAction: Equivalent of running one of the Client Action tools on a list of computers
Start-ClientCheck: Equivalent of running the client check tool on a list of computers
Start-ClientRepair: Equivalent of running the client repair tool on a list of computers
Start-RestartSMSService: Equivalent of running the restart SMS Service on a list of computers
Start-ChangeCacheSize: Equivalent of running the change cache size tool on a list of computers
Start-ClientUninstall: Equivalent of running the client uninstall tool on a list of computers
Start-ThreadedScriptBlock: Runs a scriptblock against a list of computers

All but the last one should be pretty obvious. All of the Start- cmdlets can be run with a list of computers you supply in an array or through the Get-CollectionMembers cmdlet. The Start-ThreadedScriptBlock cmdlet needs a little explaining though!

Let’s say I have the code to turn on a remote registry service on a remote machine:
<pre class="toolbar:2 lang:ps decode:true">(Get-WmiObject -Namespace 'root\cimv2' -ComputerName $ComputerName `
-Query "Select * from Win32_Service where Name like 'remoteregistry'").StartService()</pre>
So all I have to do is supply the $ComputerName variable and run this to turn on the RemoteRegistry service on a remote machine. So, how does the Start-ThreadedScriptBlock cmdlet work? It takes a script block and passes the computer name as $args[0] when it runs the script. If you want, it will also test if the computer is online first with a ping and a reverse dns lookup with the -TestConnection argument. Now, knowing this I can take my remoteregistry code and turn it into an acceptable script block for this cmdlet:
<pre class="toolbar:2 lang:ps decode:true">$ScriptBlock = {
$ComputerName = $args[0]
(Get-WmiObject -Namespace 'root\cimv2' -ComputerName $ComputerName `
-Query "Select * from Win32_Service where Name like 'remoteregistry'").StartService()
}</pre>
That’s all the editing you have to do! Wrap the script in a scriptblock and set ComputerName as args[0]. Now, let’s say I want to run this scriptblock on all the computers in my test collection. I can use the Get-CollectionMembers cmdlet to get all the computers, and then use the Start-ThreadedScriptBlock to run the cmdlet:
<pre class="toolbar:2 lang:ps decode:true">Import-Module 'C:\Program Files (x86)\Now Micro\Right Click Tools\PSModule\NowMicro-ClientActions\NowMicro-ClientActions.psm1'
 
$ScriptBlock = {
    $ComputerName = $args[0]
    (Get-WmiObject -Namespace 'root\cimv2' -ComputerName $ComputerName -Query "Select * from Win32_Service where Name like 'remoteregistry'").StartService()
}
 
$ComputerList = Get-CollectionMembers -CollectionName 'Ryans Test Collection' -Server 'nm-cm12.nowmicro.local' -SiteCode 'PS1'
Start-ThreadedScriptBlock -ComputerList $ComputerList -Script $ScriptBlock -TestConnection $true</pre>
How are the results given? A custom .Net object is returned for each computer with three properties:

Name – String that is the computer name
Result – Object that is the output of the script.
Online – Bool that is true if the computer is on or false if the computer is off

What if I wanted to get a count of all the devices online in my environment?
<pre class="toolbar:2 lang:ps decode:true">$ComputerList = Get-CollectionMembers -CollectionName 'All Systems' `
-Server 'nm-cm12' -SiteCode 'ps1'
( Start-ThreadedScriptBlock -ComputerList $ComputerList `
-Script { return $true } -TestConnection $true | Where-Object {$_.Online -eq $true} ).count
</pre>
Why did I create this? I wanted to make something that used a good threading model to run scriptblocks like I have in the right click tools. If you want more examples, consider all the other cmdlets as examples! Open up NowMicro-ClientActions.psm1 and view those functions. Almost all of them use the Start-ThreadedScriptBlock!