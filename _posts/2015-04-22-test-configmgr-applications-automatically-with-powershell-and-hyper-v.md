---
id: 66
title: Test ConfigMgr Applications Automatically with Powershell and Hyper-V
date: 2015-04-22T02:14:47+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=66
permalink: /test-configmgr-applications-automatically-with-powershell-and-hyper-v/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Application
  - Automation
  - Deployments
  - Powershell
---
I can’t take all the credit for thinking up this tool, but I’ll take the credit for making it cool! At my previous job, one of the guys there wrote a script to do the following:

Run the silent install of an application -&gt; Wait for it to complete -&gt; Copy log files to a network share -&gt; Revert VM to previous snapshot and test the next application

They could start this script and test a lot of apps overnight. I recently came across a problem where I wanted to test all the applications just created to make sure they installed correctly. We knew the silent install scripts worked outside of ConfigMgr, but the application objects were not tested. Instead of doing the work manually, I decided to write a new and improved version of the script to work with Hyper-V. Here’s the setup for the script:

1) Deploy all applications you want to test as Available to a Hyper-V VM
2) Run this script on the computer/server running Hyper-V
3) Put in the computer name (what you ping) and VM name (what you see in Hyper-V)
4) Select the applications you want to install (if you don’t see them, do machine policy updates)
5) Select if you want logs gathered and/or checkpoints made
6) Click start!

What does the script do? I made this nice flow chart to explain!
<p id="dFOhIyJ"><img class="alignnone size-full wp-image-67 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bdec295e5e.png" alt="" /></p>
Each step is logged to the same directory of the script to a file called ApplicationInstalls.log. This log file is in CMTrace format, so make sure you open it with trace!
<p id="KFvaVet"><img class="alignnone  wp-image-68 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bdee3e5c48.png" alt="" width="870" height="525" /></p>
If you choose to save checkpoints and are testing a lot of apps, make sure you have space! I used this to test 40 apps earlier today and saved a check point after each install. It ate up about 100GB of space and I had only 8GB left when the script ended!

Things to add later: Option to test uninstall after installing.

<a href="https://gallery.technet.microsoft.com/Automatically-test-d96c8ec4" target="_blank">If you want this script, you can download it here on Technet.</a>

<a href="https://github.com/Ryan2065/Ryans-Toolbox/" target="_blank">If you want to contribute any code to the script, check it out in my toolbox repo on GitHub. It’s called “Test Applications in HyperV.ps1”</a>