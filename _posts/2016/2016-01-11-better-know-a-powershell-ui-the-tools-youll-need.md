---
id: 175
title: 'Better Know a PowerShell UI: The Tools You&#8217;ll Need'
date: 2016-01-11T16:19:13+00:00
author: Ryan Ephgrave
layout: post
guid: https://www.ephingadmin.com/?p=175
permalink: /better-know-a-powershell-ui-the-tools-youll-need/
categories:
  - Powershell
  - WPF
tags:
  - ISE
  - Powershell
  - WPF
  - XAML
---
Welcome to the second edition of the long running series better know a UI! Today you’ll learn about the tools needed to create PowerShell user interfaces.

What do you need? Just Notepad and some know-how!

…I haven’t met my word quota for the day yet, so why don’t I go over some tools that make building PowerShell interfaces easier?
<p align="center"><strong>PowerShell ISE – With Add-ons!</strong></p>
<p align="left">The first tool on the list is the PowerShell ISE. The PowerShell ISE alone is a horrible user interface builder because there’s no graphical tools to build your UI and it crashes a lot when you’re working with UI code in it. Even though it has these limitations I still use the ISE because it is on every computer I go to and the crashing problem can be easily overcome with <a href="http://www.ephingadmin.com/run-wpf-powershell-scripts-in-the-ise-without-all-the-crashing/" target="_blank">this addon</a>! This add-on allows you to quickly run your code in a new PowerShell window outside of the ISE.</p>
<p align="left">A function that will make your day easier is <a title="WPF Coder!" href="http://www.ephingadmin.com/wpf-the-superduper-easy-way/" target="_blank">New-EphingWPFCode</a>. This function will parse XAML and write your UI code for you! Now all you have to do is get the XAML, which brings us to… I am continuously imporoving on it, and will soon be adding threading and class support!</p>
<p align="center"><strong>Visual Studio</strong></p>
<p align="left"><a href="https://www.visualstudio.com/products/visual-studio-community-vs" target="_blank">Visual Studio Community</a> is a free program that gives you a professional level toolset to make your user interfaces. You could also use a paid version of Visual Studio if you have MSDN, but if not the Community edition gives you all the UI building things you’ll ever need!</p>
<p align="left">That’s it. Those are the four things (PowerShell ISE, New Session Addon, New-EphingWPFCode, and Visual Studio Community) I use when I create all the user interfaces for my scripts. If you use any tools I missed, feel free to let me know in the comments!</p>
<p align="left">Join me next time when I show you how to create a PowerShell user interface to buy a duck!</p>
<p align="left"><a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/baby-duck.jpg"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border-width: 0px;" title="baby-duck" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/baby-duck_thumb.jpg" alt="baby-duck" width="244" height="184" border="0" /></a></p>
<p align="left">Until next time!</p>