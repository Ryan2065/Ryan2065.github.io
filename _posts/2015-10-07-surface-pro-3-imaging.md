---
id: 25
title: Surface Pro 3 Imaging
date: 2015-10-07T22:20:33+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=25
permalink: /surface-pro-3-imaging/
categories:
  - OSD
tags:
  - Imaging
  - Surface Pro 3
---
I have been working on a project to create a Windows 10 Task Sequence to image 800+ Surface Pro 3s. One problem we have been running into has been pen pairing. I found a <a href="http://www.windows-noob.com/forums/topic/11508-how-can-i-add-support-for-pen-pairing-during-oobe-on-a-surface-pro-3-using-system-center-2012-r2-configuration-manager/" target="_blank">great post by Niall Brady</a> about adding the OOBE pen files to your Task Sequence which will make the pen pairing OOBE menus pop up during imaging. I did some tests and the files in the link above still work on Windows 10. Great, now I have a way to do it and can move on, right? Wrong.

If you are imaging 40 Surfaces at the same time, and many of them hit the OOBE screen at once, how do you control what Surface pairs to what pen without a faraday cage? Obviously I need something to pop up before the pen OOBE menu to force the user to hit a button before pairing the pen. I was trying to figure out how to do this when I realized the EULA in the OOBE comes up before the pen pairing. So if I can just make the EULA pop up then all will be good with the world! I made a custom unattend.xml file to make skip EULA false, and tested it. Now I can move on, right? Wrong.

SCCM overwrites all the OOBE stuff to make it zero touch, so even if you specify custom settings they won’t save. Instead of abandoning my amazing idea, I decided to work around it. The unattend file is just a text file, and I can script text file edits in Powershell. I can simply change the line that says &lt;HideEULAPage&gt;true&lt;/HideEulaPage&gt; to false and move on with my life, right? Wrong.

See, in WinPE 10 (yes, they jumped from 5 to 10), Powershell doesn’t work consistently. It is a known bug that will hopefully be fixed in the near future. However, the future is not now so I have to code around this problem. I found some code somewhere (probably StackOverflow) that showed me how to use a batch file to find and replace text in a file, and modified it for this project. I created this file, created a package to add it to the task sequence, and then put it right before the OOBE step in the task sequence that is linked in the first paragraph. When I ran the task sequence, the EULA page came up and then after I hit accept, it asked me to pair the pen! Success!

Let me know if you have any issues! Here is the script:

 
<pre class="lang:ps decode:true " >@echo off &amp;setlocal
set "search=&lt;HideEULAPage&gt;true&lt;/HideEULAPage&gt;"
set "replace=&lt;HideEULAPage&gt;false&lt;/HideEULAPage&gt;"
set "textfile=C:\Windows\Panther\Unattend\unattend.xml"
set "newfile=C:\Windows\Temp\Temp-Unattend.xml"
(for /f "delims=" %%i in (%textfile%) do (
    set "line=%%i"
    setlocal enabledelayedexpansion
    set "line=!line:%search%=%replace%!"
    echo(!line!
    endlocal
))&gt;"%newfile%"
 
copy %newfile% %textfile% /Y
 
del %newfile% /q</pre> 
