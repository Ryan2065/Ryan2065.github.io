---
id: 126
title: ConfigMgr – Create Collections From Applications
date: 2014-08-15T22:01:35+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=126
permalink: /configmgr-create-collections-from-applications/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Applications
  - Collections
---
I’m living out of a hotel room this week, so I get pretty bored at night. This is good news for everyone other than me! It means I have the time to write ConfigMgr scripts I had been planning for awhile. Tonight’s script is one that allows you to create collections and deployments from applications! Yea yea, I’ll get to the import applications script soon enough…

<a href="https://gallery.technet.microsoft.com/Create-Collections-From-68d05ee2" target="_blank">Download the tool here!</a>

This script gives you the ability to quickly create install collections for your applications. Features:
<ol>
	<li>Create one application and deploy all selected applications to it, or one collection per application</li>
	<li>Quickly select the DP or DP groups you want the content in (You only need to select one DP or one DP group, or you can select ones from both columns!)</li>
	<li>Collections are automatically named the name of the application, but you can put a prefix in to help identify the collection</li>
	<li>Move the collections to a folder (Folder will be created if it doesn’t exist)</li>
	<li>Set the limiting collection</li>
	<li>Works on user or device collections</li>
	<li>Gives you all the options anyone cares about for the deployment settings.</li>
	<li>This will only work for NEW collections. This is to protect you!!!</li>
</ol>
Bugs:
<ol>
	<li>It will fail on any application that does not have content. There is a bug in Start-CMApplicationDeployment that makes it so you can’t start a deployment on an application with no content.</li>
	<li>There might be a lot of red in your console window from the tool starting content transfers. I made the decision to not check if content is already on a DP before running the cmdlet to add it to the DP. This should not cause any problems, though your console window will be very red.</li>
</ol>
<p id="kEYrKGB"><img class="alignnone size-full wp-image-127 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523b285746b.png" alt="" /></p>