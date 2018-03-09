---
id: 392
title: Script to Find ConfigMgr GUIDs
date: 2014-07-22T20:11:30+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=392
permalink: /script-to-find-configmgr-guids/
categories:
  - ConfigMgr
  - Powershell
---
 
<pre class="lang:ps decode:true " >&lt;#
Written by Ryan Ephgrave
This script puts a right click menu on everything in ConfigMgr. It is used to find the GUID of a right click menu for menu creation.
#&gt;
 
$ConfigInstallPath = $env:SMS_ADMIN_UI_PATH | Out-String
$ConfigInstallPath = $ConfigInstallPath.Trim()
$XMLPath = $ConfigInstallPath -replace "\\bin\\i386", "\XmlStorage\ConsoleRoot"
$ActionsPath = $ConfigInstallPath -replace "\\bin\\i386", "\XmlStorage\Extensions\Actions"
Get-ChildItem "$XMLPath" -Filter "*.xml" -Recurse | ForEach-Object {
	$FullFileName = $_.FullName
	$FileContent = Get-Content $FullFileName
	foreach ($line in $FileContent) {
		if ($line.ToUpper().Contains("NAMESPACEGUID=")) {
			$SplitLine = $line.Split("`"")
			$GUID = $SplitLine[1]
 
			$FilePath = "$ActionsPath\$GUID"
			New-Item -ItemType Directory -Path $FilePath -ErrorAction SilentlyContinue | Out-Null
			$strOutput = "&lt;ActionDescription Class=`"Executable`" DisplayName=`"$GUID`" MnemonicDisplayName=`"$GUID`" Description=`"$GUID`"&gt;`n"
			$strOutput = $strOutput + "&lt;ShowOn&gt;&lt;string&gt;ContextMenu&lt;/string&gt;&lt;/ShowOn&gt;`n"
			$strOutput = $strOutput + "&lt;Executable&gt;&lt;FilePath&gt;cmd.exe&lt;/FilePath&gt;`n"
			$strOutput = $strOutput + "&lt;Parameters&gt; /c Powershell.exe Add-Type -AssemblyName 'System.Windows.Forms';[Windows.Forms.Clipboard]::SetText('$GUID')&lt;/Parameters&gt;&lt;/Executable&gt;`n"
			$strOutput = $strOutput + "&lt;/ActionDescription&gt;"
			$strOutput &gt; "$FilePath\File.xml"
		}
	}
}</pre> 


At the MNSCUG meeting I showed a script I use to find the GUIDs to build right click tools. I know there are others out there, but I use my own because I added a feature to copy the GUID to your clipboard when you click on it. This lets you just paste it into a text document so you don’t have to write it down.