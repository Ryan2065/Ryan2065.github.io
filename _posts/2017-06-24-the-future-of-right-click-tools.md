---
id: 710
title: 'The future of Right Click Tools'
date: 2017-6-24T01:10:39+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=710
permalink: /the-future-of-right-click-tools/
categories:
  - PowerShell
  - ConfigMgr
  - SCCM
---

# The future of Right Click Tools

You may have missed the announcement, but buried in the documentation
for the new SCCM Tech Preview (1706) was about a new feature to create
and run PowerShell scripts from the ConfigMgr console. How is this 
different than Applications and CIs? These scripts run on demand from a 
right click context menu, and use fast channel to run and give feedback instantly!

ConfigMgr finally has built in, extendable, right click tools!

Note, this a tech preview, so don't expect to use this post as a guide for
how to set it up once it's released!

Let's go through creating and approving your first script...

## Create the script

Let's start with an easy reboot script:

```
shutdown /r /t 10 /f
```

In the console, go to Software -> Scripts, right click and choose Create Script

![CreateScript](\images\2017-6-24\CreateScript.jpg)

Name the script and copy it into the script window:

![NameScript](\images\2017-6-24\NameScript.jpg)

Click Next -> Next -> Finish.  Now we have a script, what do we do with it? 

If you look at the script in the console, it is in the state "Waiting for approval." This means the script cannot be run in the environment until someone
approves it! This TP also has the first view into how approvals will work in SCCM
in the future! You can configure approvals so the author cannot approve their work, which allows you to add a second pair of eyes on all scripts in your environment! Cool, right?

If you just installed this TP, you should have permissions to approve the script. In the upper right, click Approve/Deny:

![ApproveDeny](\images\2017-6-24\ApproveDeny.jpg)

In the wizard, make sure you select Approve:

![Approve](\images\2017-6-24\Approve.jpg)

Now that the scirpt is approved, we can run it on demand on remote machines. How cool is that?

Go to Assets and Compliance, click on Device Collections, Right click on a collection you want to run this on, and select "Run Script"

![RunScript](\images\2017-6-24\RunScript.jpg)

Now, select the Reboot Computer script, and Next -> Next -> Finish:

![RunScript2](\images\2017-6-24\RunScript2.jpg)

In 15 seconds, my Windows 10 computers restarted! This means it took 5 seconds for the computer to get the new policy and run the script!

Right click tools are finally in SCCM!

![Yay](https://media.giphy.com/media/f2fX7GtXh1nbi/giphy.gif)