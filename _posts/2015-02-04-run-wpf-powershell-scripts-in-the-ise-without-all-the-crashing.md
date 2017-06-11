---
id: 110
title: Run WPF Powershell Scripts In The ISE Without All The Crashing!
date: 2015-02-04T17:59:58+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=110
permalink: /run-wpf-powershell-scripts-in-the-ise-without-all-the-crashing/
categories:
  - Powershell
tags:
  - ISE
  - Powershell ISE
  - WPF
---
One of the biggest problems (for me at least) with the Powershell ISE is the fact that all scripts have to run through the ISE. If you get into heavy WPF writing, you’ll find that this causes the ISE to get all crashy every now and then. For years this made me use Power GUI. Recently, Power GUI started getting slow for some of my larger (1000+ line) scripts. I decided it was time to switch back to ISE and fix the crash problem.

For those of you who use Power GUI, you know there is a cool feature that lets you run the script in a new window. I wrote an extension to the ISE to add in this feature. To add an extension to ISE, you’ll want to create a .ps1 file called “Microsoft.PowerShellISE_profile.ps1” and put it in your <a href="http://www.howtogeek.com/50236/customizing-your-powershell-profile/" target="_blank">Powershell Profile</a> (I have it in Documents\WindowsPowershell). Once the file is created, copy and paste this code into the file and click save. This will add a button with the hot key Alt+F5 to open a script in a new window.
<pre class="lang:ps decode:true">$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Run In New Window", {
    If ($psISE.CurrentFile.IsUntitled) {
        $ScriptBlock = $psISE.CurrentFile.Editor.Text
        $newGuid = [guid]::NewGuid()
        $TempFilePath = "$env:TEMP\$newGuid.ps1"
        $ScriptBlock &gt; $TempFilePath
        $Command = "$TempFilePath"
        Start-Process Powershell.exe -Wait -ArgumentList "-noexit","-ExecutionPolicy Bypass","-file $Command"
        Remove-Item -Path $TempFilePath -Force
    }
    else {
        $psISE.CurrentFile.save()
        $CurrentFile = $psIse.CurrentFile
        $CurrentFilePath = $CurrentFile.FullPath
        Start-Process Powershell.exe -Wait -ArgumentList "-noexit","-ExecutionPolicy Bypass","-file `"$CurrentFilePath`""
    }
},"ALT+F5") | out-Null</pre>
This will copy all the text in your active Powershell ISE tab and put it in a new powershell script in your temp folder. It will wait for the script to end and then delete the file it created. You will not be able to run anything in the ISE until the script window is closed.

Let me know if you have any issues with this!