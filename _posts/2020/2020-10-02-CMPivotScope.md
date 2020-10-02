---
id: 723
title: 'CMPivot - How to change the scope'
date: 2020-10-02
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=723
permalink: /EFPoshHowIUseIt/
categories:
    - PowerShell
    - MEMCM
---

# CMPivot - Changing the scope

CMPivot is cool and all, but you are required to have the Default security scope in order to use it. This really makes it hard to get your users to adopt it if you have RBAC set up the way Microsoft has been asking us to do it for forever.

"Here's a great tool for your helpdesk, just let them see everything!"

![ComeOn](https://www.ephingadmin.com/images/2020/ComeOn.gif)

So I've played around with it on the backend, and have come up with a solution that lets you select what SCOPEs can run CMPivot!

First, the proof!

I created a SCOPE called CMPivotDemo and assigned a user to that scope, removing Default.  I then tried to run CMPivot:

![CantRunCMPivot](https://www.ephingadmin.com/images/2020/CantRunCMPivot.jpg)

As we all know, CMPivot is a fancy UI on top of a CM Script. If you just assign the CM Script called "CMPivot" to the scope your user is in, they can then launch CMPivot! This is achievable through PowerShell:

``` PowerShell
$Script = Get-CMScript -ScriptName 'CMPivot' -Fast
Add-CMObjectSecurityScope -Name 'CMPivotDemo' -InputObject $Script
```

In your environment, just change ```CMPivotDemo``` to the name of the scope you want to access CMPivot. Run the script and...

![CMPivotWorks](https://www.ephingadmin.com/images/2020/CMPivotWorks.gif)

