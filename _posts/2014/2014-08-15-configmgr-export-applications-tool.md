---
id: 122
title: 'ConfigMgr: Export Applications Tool'
date: 2014-08-15T21:59:15+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=122
permalink: /configmgr-export-applications-tool/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Applications
  - Tools
---
I created a tool to make exporting ConfigMgr applications easier.

<img class="alignnone size-full wp-image-123 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523a955be3e.png" alt="" />

This should be pretty self-explanatory. Put in CM Server Name and CM Site Code and then click Load Applications. This will bring up a list of all the applications you have on that site. Select the applications you want to export (multi-select is enabled) and then fill out the fields. If you check “Export as one file per application selected” you will need to supply a directory in the Save Path. If you don’t check it, you’ll need to supply a zip filename/path. Once that’s done, click start and watch the applications export!

Next on the agenda, write a tool to import applications and move the content to a different directory!

<a href="https://gallery.technet.microsoft.com/Export-Applications-The-71877c54" target="_blank">Download File…</a>