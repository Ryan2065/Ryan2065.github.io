---
id: 74
title: Powershell Right Click Tools – Beta 1.5 available
date: 2013-08-26T14:50:50+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=74
permalink: /powershell-right-click-tools-beta-1-5-available/
categories:
  - Right Click Tools
tags:
  - ConfigMgr
  - Console Extension
---
Beta 1.5 is released!  I made sure all the old tools work, so the only beta parts are the new tools. This beta version includes:
<p align="left">Revision History – Will show up on Applications
Content Status – Will show up on Applications, Packages, Software Update Packages, Driver Packages, Boot Images, and OS Images
User Devices – Will show up on Users
Client Information – Will show up on Devices</p>
Some information in System Information was moved to Client Information (Execution history and Applications).  A cache tab was added to System Information that isn’t complete yet. Everything else in this tool should work.

There is also an update check which will check for updates once a day. If you don’t want it to check for updates, go to the tool install directory and remove the line “AutoUpdate=True” from “Tool Properties.ini”

This is a beta so if you find any errors, please contact me and let me know! I’ll be putting out more frequent updates (hoping for weekly updates) while this is in beta. These updates will add features and fix bugs.