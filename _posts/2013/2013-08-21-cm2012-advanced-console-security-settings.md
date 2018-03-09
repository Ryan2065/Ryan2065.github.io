---
id: 89
title: CM2012 – “Advanced” Console Security Settings
date: 2013-08-21T15:15:36+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=89
permalink: /cm2012-advanced-console-security-settings/
categories:
  - ConfigMgr
  - Right Click Tools
tags:
  - Comment
  - Console
  - XML
---
Let’s pretend you’re a CM2012 admin (far-fetched?) and you have a group of people packaging for you. You’ve asked them time and time again to not delete an application’s revision history, but they just won’t listen. So, how do you keep them from accessing an application’s revision history, but still allow them to package and deploy applications?

The menus in the CM2012 console are all generated through a few XML files stored in
%InstallDirectory%XmlStorageConsoleRoot
You can modify the XML files to change certain properties of the console, like what options you will see. If you want to completely get rid of Revision History from the console, open the XML file associated with the Software Library (SoftwareLibraryNode.xml). I’m going to use XML Notepad because it makes this so much easier, but you can use whatever program you want to. In my guide <a href="http://ephingadmin.com/wp/create-your-own-right-click-tools-part-1/" target="_blank">Create Your Own Right Click Tools – Part 1</a>, I went through how to navigate XML files. I won’t repeat all of that, so if you want to learn, check that guide.

To quickly find the Revision History section, CTRL+F for Revision:
<p id="yWeoHxL"><img class="alignnone size-full wp-image-90 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c956974a83.png" alt="" /></p>
This shows you the Revision History section. Highlight ActionDescription (the node this is a part of), Right click –&gt; Change To –&gt; Comment:

&nbsp;
<p id="SXIlrAP"><img class="alignnone size-full wp-image-92 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c95c8e16a8.png" alt="" /></p>
<p id="aTcooPJ"><img class="alignnone size-full wp-image-93 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c95df6f14f.png" alt="" /></p>
Now, it’s a comment.  Save the document, open up your console (if it was already open, re-open it) and Revision History will be gone!

&nbsp;
<p id="tkipSlO"><img class="alignnone size-full wp-image-94 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c95fcd8133.png" alt="" /></p>
&nbsp;