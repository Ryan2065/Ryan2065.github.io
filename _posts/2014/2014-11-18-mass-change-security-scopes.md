---
id: 139
title: Mass Change Security Scopes
date: 2014-11-18T22:11:44+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=139
permalink: /mass-change-security-scopes/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Scopes
  - Tools
---
At MMS someone mentioned there was no good way to mass change scopes in an environment. You have to click on each item and edit the scope in order to change it. I’ve never been a fan of clicking that much, so I decided to make a tool to make it easy to change scopes!

<img class="alignnone size-full wp-image-140 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523d9195689.png" alt="" />

With this tool, simply put in the server name and site code of your site and hit connect. Then, select a scope and a class from the dropdown list to view all available items in the class. If the item is in the scope it will show up on the right side, and if not it will show up on the left side. Right click on any item (or multiple) and you will be given the option to either add it to a scope or remove it from a scope.

The console window is the log window, so watch that for information about what is happening when the window is not responding. Also, classes can be a little confusing. I’m using what is in WMI, so they are not extremely friendly. Also, some things are grouped differently than in the console. For instance, under SMS_Queries, you’ll find custom queries and status message queries.

These are the classes you’ll be able to change:

SMS_Application
SMS_BootImagePackage
SMS_BoundaryGroup
SMS_ClientSettings
SMS_ConfigurationItem
SMS_ConfigurationPolicy
SMS_DistributionPointGroup
SMS_DistributionPointInfo
SMS_DriverPackage
SMS_GlobalCondition
SMS_ImagePackage
SMS_MigrationJob
SMS_OperatingSystemInstallPackage
SMS_Package
SMS_Query
SMS_Site
SMS_SoftwareUpdatesPackage
SMS_TaskSequencePackage

If you have any problems or I missed a class let me know!

Download: <a href="https://gallery.technet.microsoft.com/Mass-Change-Security-Scopes-db9efcd8" target="_blank">Change Scopes</a>