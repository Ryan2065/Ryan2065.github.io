---
id: 15
title: Create Your Own Right Click Tools – Part 3
date: 2013-08-11T02:16:57+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=15
permalink: /create-your-own-right-click-tools-part-3/
categories:
  - Right Click Tools
---
In <a href="http://ephingadmin.com/wp/create-your-own-right-click-tools-part-1/" target="_blank">Part 1</a> of this series, I showed you how to find the GUID of the menu. In <a href="http://ephingadmin.com/wp/create-your-own-right-click-tools-part-2/" target="_blank">Part 2</a>, I showed you how to create a menu item and run a script. Now, I’ll list some tips I’ve found as I’ve worked on the <a href="http://psrightclicktools.codeplex.com/" target="_blank">Powershell Right Click Tools</a>.

<a href="http://ephingadmin.com/create-your-own-right-click-tools-part-3/#Add Sub Menus">1. Add Sub Menus</a>
<a href="http://ephingadmin.com/create-your-own-right-click-tools-part-3/#Run Powershell Scripts">2. Run Powershell Scripts</a>

<a id="Add Sub Menus"></a>1. Add Sub Menus

I’ve taken the “Content Status” menu option we created in Part 2, and added it to the group “Test Group”
 
<pre class="lang:ps decode:true " >&lt;ActionDescription Class="Group" DisplayName="Test Group"&gt;
	&lt;ShowOn&gt;&lt;string&gt;ContextMenu&lt;/string&gt;&lt;/ShowOn&gt;
	&lt;ActionGroups&gt;
 
		&lt;ActionDescription Class="Executable" DisplayName="Content Status"&gt;
		&lt;ShowOn&gt;&lt;string&gt;ContextMenu&lt;/string&gt;&lt;/ShowOn&gt;
		&lt;Executable&gt;
			&lt;FilePath&gt;Powershell.exe&lt;/FilePath&gt;
			&lt;Parameters&gt;-sta -windowstyle hidden -executionpolicy bypass -file "C:Content Status.ps1"&lt;/Parameters&gt;
		&lt;/Executable&gt;
		&lt;/ActionDescription&gt;
	
	&lt;/ActionGroups&gt;
&lt;/ActionDescription&gt;</pre> 

&nbsp;

Line 1 creates the group name “Test Group”.  Line two tells the console where to display this new group.  Line three starts the ActionGroups, and anything under here will show up in the menu.  As you can see, I put my Content Status menu executable under here.  Now, when I open the console, I see Content Status in the menu Test Group.
<p id="QSFKeSH"><img class="alignnone size-full wp-image-20 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564a8e681c561.png" alt="" /></p>
And that’s how you’d create sub menus!  You can even create sub menus of sub menus of sub menus if you wanted. I don’t know if there is a limit to how deep you can go.

<a href="http://ephingadmin.com/create-your-own-right-click-tools-part-3/#Run Powershell Scripts">2. Run Powershell Scripts</a>

Powershell scripts don’t run straight out of the box like VB scripts do.  To run a VB script, you just need to put wscript.exe as your executable, and then the script name as your parameter. If you tried this with Powershell, it wouldn’t work! Why is this?  Execution policy!

The odd thing about the ConfigMgr 2012 console is the execution policy is different than the computer’s execution policy. To illustrate this point, I created a right click tool that simply runs a batch file to read the powershell execution policy. Here is a snap shot. The window on the left was run file normally to get the computer’s execution policy, and the right was run through a right click tool to get ConfigMgr’s execution policy:
<p id="pOCveGx"><img class="alignnone size-full wp-image-21 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564a8e7c2c4c8.png" alt="" /></p>
ConfigMgr’s execution policy doesn’t mirror the execution policy of the computer. So, how do I launch all my Powershell Right Click Tools? I use a number of switches to force them to display.  The first switch is –ExecutionPolicy, which I set to bypass. I then also set –WindowStyle Hidden and –Sta when I’m running a Windows Forms script, to hide the ugly console window. To launch a powershell windows forms script with no arguments, here is the parameters line: