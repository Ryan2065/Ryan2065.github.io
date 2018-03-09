---
id: 34
title: Add help text to your Powershell functions!
date: 2015-09-28T22:40:42+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=34
permalink: /add-help-text-to-your-powershell-functions/
categories:
  - ISE
  - Powershell
tags:
  - Get-Help
  - ISE
  - Powershell
---
On Twitter I asked if there was an ISE addon to put help text into a function. The response led me to <a href="https://www.powershellgallery.com/packages/PowerShellISEModule/" target="_blank">something Ed Wilson wrote</a>that simply added in help text at the line you were on. I wanted something that parsed my current function and added in help text for the parameters, so I decided to use that function as a starting point and expand on it.

Here’s how my new ISE addon works. First, make sure the current version of the function is loaded into the session by highlighting it and hitting F8. Then, highlight the parameters of the function and then hit Alt+F6. What this script does is it compares the text of your parameters to the text of all functions loaded into the current session. If it finds a match, it generates the help text and automatically adds the parameter text. If you don’t want to do all of that, you can simply hit Alt+F6 without highlighting anything and it will add in help text without any parameter information.

Let me know if there are any problems! Simply load this into your Powershell ISE profile script (c:\users\username\documents\WindowsPowershell\Microsoft.PowershellISE_Profile.ps1 is the default) and restart the ISE session. Key command is Alt+F6 but can be changed pretty easily by editing the last line.
<pre class="lang:ps decode:true">$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Add-Help", {
&lt;#
    .SYNOPSIS
        Adds in help comments
 
    .DESCRIPTION
        Highlight the parameters of a function and hit Alt+F6 to add in help comments
        Based off the Add-Help function in the PowershellISEModule by Ed Wilson
        https://www.powershellgallery.com/packages/PowerShellISEModule/
 
    .NOTES
        AUTHOR: Ryan Ephgrave
        LASTEDIT: 09/28/2015 14:51:13
 
   .LINK
        http://ephingadmin.com
#&gt;
 
$helpText = @"
&lt;#
    .SYNOPSIS
        
 
    .DESCRIPTION
        
   
    .EXAMPLE
        
 
"@
 
    $SelectedText = $psise.CurrentFile.Editor.SelectedText
    $Functions = Get-ChildItem Function:
    $found = $false
    Foreach ($Function in $Functions) {
        if ($Function.ScriptBlock.ToString().Contains($SelectedText) -and ($found -ne $true)) {
            $found = $true
            $Function.ParameterSets | ForEach-Object { 
                $Parameters = $_.Parameters.Name
                foreach ($Parameter in $Parameters) {
                    $helpText = $helpText + "`n    .PARAMETER $Parameter`n        `n"
                }
             }
        }
    }
 
$helpText = $helpText + @"   
 
    .NOTES
        AUTHOR: 
        LASTEDIT: $(Get-Date)
 
   .LINK
        
#&gt;
 
"@
$helpText = $helpText + $SelectedText
$psise.CurrentFile.Editor.InsertText($helpText)
},"ALT+F6") | out-Null</pre>