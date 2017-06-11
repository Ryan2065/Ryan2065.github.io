---
id: 76
title: 'Right Click Tool Preview – Revision History &#038; Content Status'
date: 2013-08-16T14:51:39+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=76
permalink: /right-click-tool-preview-revision-history-content-status/
categories:
  - Right Click Tools
tags:
  - ConfigMgr
  - Console Extension
---
All of the old right click tools are more geared towards managing computers. I wanted to expand this next set, so you’ll be finding tools all over the place. I’m showing off two new tools that will pop up when you right click on Applications…
<p align="center">Revision History</p>
<p align="left">While the standard Revision History tool works well, it is hard to see what changed between two revisions. I came up with a revision history tool that gives you a list of revisions on the right side, so you can quickly change between them and easily see what changed!  There is a menu at the top which will also let you delete or restore a revision, just like the built in tool.  I opened up this tool on my 7-Zip application and looked at Revision 1:</p>
<p id="uXSVNIK"><img class="alignnone size-full wp-image-77 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c90a32e7ac.png" alt="" /></p>
While this installs 7-Zip, all the comments are random things to help me figure out which fields go to which XML property. I want to know what changed between Revision 1 and Revision 11, I just click on 11 and it switches to Revision 11, and the fields that are different all update with the correct information.
<p id="EbwFXdp"><img class="alignnone size-full wp-image-78 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c90d3ab480.png" alt="" /></p>
<p align="left">This tool is still in development, and I may add something at the bottom to tell you how many fields changed between the two revisions, broken down by tabs. I’ve got a lot on my plate though, so that might not make this release.</p>
<p align="center">Content Status</p>
<p align="left">I hate how you have to switch to the Monitoring tab to get detailed information about the content.  This tool will actually show up on Applications, Packages, Software Update Packages, Driver Packages, Boot Images, and Operating System Images, so it isn’t limited to just Applications.</p>
<p align="left">This one is very straightforward… Right click on the object, go to Content Status, and then you’ll see this box:</p>
<p id="RgHzaMm"><img class="alignnone size-full wp-image-79 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c910c61653.png" alt="" /></p>
<p align="left">Here you’ll see content information (size, % complete, etc), what DP groups it is a member of, and what DPs it is on.  You’ll also see the status of the content on each DP, so you know which ones it fails on.  Finally, you will be able to right click on any DP or DP Group and validate, redistribute, or remove content.</p>
<p align="left">Well, that’s it for tonight!  I hope you all have a great weekend and I’ll be back Monday with some more updates.</p>