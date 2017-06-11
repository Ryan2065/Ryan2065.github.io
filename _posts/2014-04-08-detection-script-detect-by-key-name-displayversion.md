---
id: 344
title: 'Detection Script: Detect by Key Name &#038; DisplayVersion'
date: 2014-04-08T19:53:46+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=344
permalink: /detection-script-detect-by-key-name-displayversion/
categories:
  - ConfigMgr
  - Powershell
---
I had a coworker come across an interesting problem with an application he was trying to detect. They decided to be different in how they named the uninstall registry keys, which makes it difficult to find. In the key, their product name &amp; version were listed instead of a GUID. So we were stuck looking for a registry key that could look like HKLM\Software\Microsoft\Windows\CurrentVersion\Uninstall\Product-1234. My coworker wanted an easy way to detect the application that would work for future versions as well. So he asked me to help him script it out in Powershell.

First off, whenever I use a detection script that checks the registry, I always suppress error messages. If an error occurs, it returns “something” so it marks the application as found. I don’t want this, so I put this at the start of my script:
<pre class="lang:ps decode:true ">$ErrorActionPreference = "SilentlyContinue"</pre>
Now, I want this script to be easily updated with the newer version number for new releases, so I specify the version at the top of the script using the Version data type. I am using the Version data type because versions are stored in the x.x.x.x format, which aren’t very easy to compare if that data type isn’t used.
<pre class="lang:ps decode:true ">[Version]$CompareVersion = "1.0.0"</pre>
Now, I know the registry key name is always going to be ProductName-Version, so I just want to search registry keys that start with ProductName. I can do that with this command:
<pre class="lang:ps decode:true ">Get-Item -Path Registry::"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\ProductName*" | ForEach-Object {</pre>
After this, it is a simple matter of loading the properties of the key, grabbing the DisplayVersion field, and then comparing the two. The script returns true if DisplayVersion = CompareVersion.
<pre class="lang:ps decode:true ">    $Properties = Get-ItemProperty $_.PSPath
	$Version = $Properties.DisplayVersion
	[Version]$Version = $Version.Trim()
	If ($Version -eq $CompareVersion) {return $True}
}</pre>
And here is the script all put together:
<pre class="lang:ps decode:true ">$ErrorActionPreference = "SilentlyContinue"
[Version]$CompareVersion = "1.0.0"
Get-Item -Path Registry::"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\ProductName*" | ForEach-Object {
    $Properties = Get-ItemProperty $_.PSPath
	$Version = $Properties.DisplayVersion
	[Version]$Version = $Version.Trim()
	If ($Version -eq $CompareVersion) {return $True}
}</pre>