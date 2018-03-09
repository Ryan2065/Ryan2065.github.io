---
id: 36
title: Find Devices With No Boundary
date: 2015-09-21T22:43:53+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=36
permalink: /find-devices-with-no-boundary/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Boundaries
  - ConfigMgr
  - Powershell
---
Ever wonder if all of your devices are in a boundary? Six months ago I wrote a script to help solve this problem. I told no one about it and forgot about it until someone on the mailing list mentioned that they made a SQL query to do something similar. Not to be outdone, I decided to share my script with the world!

This script is pretty simple to run. It only needs the parameters SiteServer and SiteCode. You can pipe the output to GridView with Out-GridView to make the data easier to read also! This is what the output looks like once you run the script with | Out-GridView:
<p id="KmdluaW"><img class="alignnone size-full wp-image-47 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bafcd0ad46.png" alt="" /></p>
Well, your output won’t have those black lines, but you get the idea. This gives you a list of every computer in your organization and shows you which ones are not in a boundary. Let me know if you have any questions!

<a href="https://github.com/Ryan2065/Powershell-Scripts/blob/master/FindDevicesNoBoundary.ps1" target="_blank">View the script on GitHub here!</a>