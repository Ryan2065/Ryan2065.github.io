---
id: 352
title: 'OSD: Move Computers in AD Auto Magically'
date: 2014-04-17T19:58:23+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=352
permalink: /osd-move-computers-in-ad-auto-magically/
categories:
  - ConfigMgr
---
Here is a script I wrote to put computers into different OUs based on the computer name.

 <pre class="lang:vb decode:true " >On Error Resume Next
 
Set OSDenv = CreateObject("Microsoft.SMS.TSEnvironment")
 
SetOuByCompName "Test","LDAP://OU=TestOSD,DC=EphingDomain,DC=org"
 
Function SetOUByCompName (strNameStartsWith, strOU)
    strCompName = OSDenv("OSDComputerName")
	If Instr(1, strCompName, strNameStartsWith, 1) = 1 Then OSDenv("OSDDomainOUName") = strOU
End Function</pre> 


In this script, if the computer’s name starts with Test in OSD, it will change the OU this computer goes in to to <a href="ldap://OU=TestOSD,DC=EphingDomain,DC=org">LDAP://OU=TestOSD,DC=EphingDomain,DC=org</a>

You can add as many matches you want by creating more of these lines:

 
<pre class="lang:vb decode:true " >SetOuByCompName "Test","LDAP://OU=TestOSD,DC=EphingDomain,DC=org"</pre> 


In order to set this up in ConfigMgr, create a Package with no program that has a folder with the script in it as the source folder, like this:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image7-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image7 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image7-1_thumb.png" alt="image7 (1)" width="359" height="365" border="0" /></a>

And then create a new “Run Command Line” step in your task sequence, sometime after the partition disk step, but before the Apply Network Settings step. Configure it to look like this:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image8-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image8 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image8-1_thumb.png" alt="image8 (1)" width="369" height="334" border="0" /></a>

Now you’re all set! If you change the script, make sure you update the content on the package so your DPs and task sequences get the change!