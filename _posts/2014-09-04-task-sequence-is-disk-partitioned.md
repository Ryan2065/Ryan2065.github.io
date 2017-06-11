---
id: 132
title: Task Sequence–Is Disk Partitioned?
date: 2014-09-04T22:10:15+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=132
permalink: /task-sequence-is-disk-partitioned/
categories:
  - ConfigMgr
tags:
  - Partitioned
  - Task Sequence
---
One of the problems with using a number of custom front ends for OSD is it forces the user to wait until after the disk is partitioned to pop up. I understand the reason for this, in order to run a package ConfigMgr needs to put the files somewhere, but that doesn’t mean I like it!

I’ve come up with what I believe is a fairly simple way to skip this if the disk is already partitioned (so a refresh scenario). The idea is I put a run command line step before the Partition Disk steps which calls a script in a package and it is set to continue on error. The script simply sets two OSD variables, one is OSDSkipPartitioning and the other is OSDSystemDrive. Then, set your Partition Disk steps to only run if OSDSkipPartitioning does not equal true. OSDSystemDrive is set so you can use that to apply the image to the right drive letter.

So what happens? If the disk is not already partitioned, the run command line step will fail because it can not download the content. Since it is set to continue on error, it ignores this. The partition disk steps will then run because OSDSkipPartitioning is not set. Those partition disk steps should set the OSDSystemDrive variable and then the apply image step will apply the image to OSDSystemDrive.

If the disk is already partitioned, the run command line step will complete and set the two variables. The partition disk steps will not run because OSDSkipPartitioning is set to true and the image will be applied to OSDSystemDrive.

You might want to test this out in your environment before you do anything. I could see the OSDSystemDrive being potentially set to the wrong drive letter depending on how the partitions are already set on the disk. It is set based on where ConfigMgr decided to download the package to. In my experiments, the system drive was C: (so the 350mb partition) and the primary drive was D:. This correctly set OSDSystemDrive to D:, and after it was restarted into the OS the drive was the correct C:.

Anyway, enough with all the explanation, here are the steps!

1) Create a VB script called whatever you want (I called it SkipPartitioning.vbs) and the script is this:
<pre class="lang:vb decode:true">On Error Resume Next
Set OSDenv = CreateObject("Microsoft.SMS.TSEnvironment")
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objShell = CreateObject("Wscript.Shell")
 
OSDenv("OSDSkipPartitioning") = "true"
strPath = objShell.CurrentDirectory
strDrive = objFSO.GetDriveName(strPath)
OSDenv("OSDSystemDrive") = strDrive</pre>
2) Create a Package called whatever you want (I called mine OSD – Skip Partitioning) and has the script folder as it’s content. No program is needed!
<p id="lphSAwv"><img class="alignnone size-full wp-image-133 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523caabad9c.png" alt="" /></p>
3) Add a Run Command Line step to the task sequence to run right after “Restart in Windows PE”. Reference the package you just created and the command line is cscript SkipPartitioning.vbs Make sure you check “Continue on error” in the options tab!

&nbsp;
<p id="xdDGKIn"><img class="alignnone size-full wp-image-134 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523cc74937c.png" alt="" /></p>
4) Add a condition to your partition disk steps of OSDSkipPartitioning not equals “true”. Also, on your partition disk steps, make sure the primary partition sets the variable OSDSystemDrive:

&nbsp;
<p id="cBsdzmH"><img class="alignnone size-full wp-image-135 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523ce5d0d3a.png" alt="" /></p>
<p id="aCqNOYF"><img class="alignnone size-full wp-image-136 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523cff17f31.png" alt="" /></p>
5) Go to your apply image step/steps and change the destination to the variable OSDSystemDrive:

&nbsp;
<p id="SXCMGjm"><img class="alignnone size-full wp-image-137 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523d1cf069a.png" alt="" /></p>
And you’re done! It should be working now.

&nbsp;