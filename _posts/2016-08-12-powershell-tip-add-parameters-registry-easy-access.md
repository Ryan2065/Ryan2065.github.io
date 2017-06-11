---
id: 645
title: 'PowerShell Tip: Add Parameters to Registry for Easy Access'
date: 2016-08-12T00:37:46+00:00
author: ryan2065@gmail.com
layout: single
guid: http://www.ephingadmin.com/?p=645
permalink: /powershell-tip-add-parameters-registry-easy-access/
categories:
  - Powershell
tags:
  - Parameters
  - Powershell
  - Registry
---
I tend to work on large PowerShell scripts that accept parameters. In the past I’ve always defined these parameters in the script while I test, and then I have to go back and remove them when I blog about the script here. I decided I’d figure out an easier way to do this!

I’m already using my <a href="http://www.ephingadmin.com/run-wpf-powershell-scripts-in-the-ise-without-all-the-crashing/" target="_blank">ISE addon to run scripts in a new PowerShell session</a>, so I decided to edit that to add parameter support!

First off, I only care to do this for saved scripts, so I only have to edit the “else” statement in my ISE addon:

 
{% highlight powershell linenos %}
$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Run In New Window", {
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
},"ALT+F5") | out-Null
{% endhighlight %}


I ended up deciding to store these variables in the registry. I post a lot of my projects to GitHub and didn’t want a file full of variables in my Git repo. So I went to my registry and created a new Key under HKCU:\Software called EphingParams. I then need a way to tell it what script I’m using. The current project I’m working on has a lot of scripts in the same directory all using a similar set of parameters. So I created a key with the name of the folder “ConfigMgr_Console_Extensions”

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/08/image.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/08/image_thumb.png" alt="image" width="242" height="48" border="0" /></a>

The idea is, my launch script will check to see what the file path is. If the file path includes “\ConfigMgr_Console_Extensions\” (meaning it is in that folder) or the file path includes “\ConfigMgr_Console_Extensions.” (meaning it is the file name) then it will grab the parameters. Now I just need to use the registry editor to make a few test parameters:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/08/image-1.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/08/image_thumb-1.png" alt="image" width="244" height="63" border="0" /></a>

I’ve finished with the setup! Now I need to add the logic to my ISE addon:

 
<pre class="lang:ps decode:true " >$arguments = ''
try {
    Get-ChildItem HKCU:\Software\EphingParams | ForEach-Object {
        $Name = $_.PSChildName
        $NameToLower = $Name.ToLower()
        if(($CurrentFilePath.ToLower().Contains("\$nametolower\")) -or ($CurrentFilePath.ToLower().Contains("\$nametolower."))) {
            $Script:Properties = Get-ItemProperty -Path $_.PSPath
            $PropertyMembers = Get-Member -InputObject $Properties
            foreach ($property in $PropertyMembers) {
                if($Property.MemberType -eq 'NoteProperty') {
                    if(($Property.Name -ne 'PSChildName') -and ($Property.Name -ne 'PSParentPath') -and ($Property.Name -ne 'PSPath') -and ($Property.Name -ne 'PSProvider')) {
                        $PropName = $Property.Name
                        $arguments = $arguments + " -$($Property.Name) `"$($Properties.$PropName)`""
                    }
                }
            }
        }
    }
}
catch {
    $arguments = ''
}</pre> 


This script block will get all registry keys in HKCU:\Software\EphingParams, and then check the filepath to see if it includes a child registry key. If it does, it grabs all the properties (excluding the 4 default properties) and adds them to the launch code as  -PropertyName “PropertyValue”. The end result is your script being executed with the parameters in the registry!

Here’s the full script:

 
<pre class="lang:ps decode:true " >$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Run In New Window", {
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
        $arguments = ''
        try {
            Get-ChildItem HKCU:\Software\EphingParams | ForEach-Object {
                $Name = $_.PSChildName
                $NameToLower = $Name.ToLower()
                if(($CurrentFilePath.ToLower().Contains("\$nametolower\")) -or ($CurrentFilePath.ToLower().Contains("\$nametolower."))) {
                    $Script:Properties = Get-ItemProperty -Path $_.PSPath
                    $PropertyMembers = Get-Member -InputObject $Properties
                    foreach ($property in $PropertyMembers) {
                        if($Property.MemberType -eq 'NoteProperty') {
                            if(($Property.Name -ne 'PSChildName') -and ($Property.Name -ne 'PSParentPath') -and ($Property.Name -ne 'PSPath') -and ($Property.Name -ne 'PSProvider')) {
                                $PropName = $Property.Name
                                $arguments = $arguments + " -$($Property.Name) `"$($Properties.$PropName)`""
                            }
                        }
                    }
                }
            }
        }
        catch {
            $arguments = ''
        }
        Start-Process Powershell.exe -Wait -ArgumentList "-noexit","-ExecutionPolicy Bypass","-file `"$CurrentFilePath`" $arguments"
    }
},"ALT+F5") | out-Null</pre> 
