---
id: 10
title: Create Your Own Right Click Tools – Part 1
date: 2013-07-07T01:59:36+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=10
permalink: /create-your-own-right-click-tools-part-1/
categories:
  - Right Click Tools
---
Have you ever right clicked on an object in ConfigMgr and not found what you were looking for?  Well ConfigMgr gives you the ability to add options in their right click menus.  This series will teach you how to create your very own right click tool!
<p align="center">Part 1 – Dive into XML!</p>
<p align="left">All nodes and menus in CM2012 have GUIDs associated with them. In order to create a right click tool in the console, you first need to know the GUID of the menu item you want to add it on. You’ll also want to know which WQL class the information is pulled from if you want to pass variables to your tool.</p>
<p align="left">First off, you’ll need <a href="http://www.microsoft.com/en-us/download/details.aspx?id=7973" target="_blank">XML Notepad</a>, a computer with the ConfigMgr console installed on it, and an idea of where you want this tool.  Recently, I came up with a new Content Status script, and thought it would be nice to have that as a right click option on applications, so you could quickly see where the content was, not just the percent complete.</p>
<p align="left">XML Notepad isn’t required, but I don’t think reading raw XML data is fun so I’m sticking with what’s easy.  Open up XML Notepad and navigate to your ConfigMgr XML directory.  For the default install, it’s:
C:Program Files (x86)Microsoft Configuration ManagerAdminConsoleXMLStorageConsoleRoot</p>
Here you have a number of XML files that represent each node of the console.
Assets and Compliance – AssetManagementNode.xml
Software Library – SoftwareLibraryNode.xml
Monitoring – MonitoringNode.xml
Administration – SiteConfigurationNode.xml

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/06/image1.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/06/image_thumb1.png" alt="image" width="754" height="504" border="0" /></a>

Since I want to put a right click tool in the Software Library node, open SoftwareLibraryNode.xml, and you should see this in XML Notepad:

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image3.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb3.png" alt="image" width="827" height="274" border="0" /></a>

Now, I know you’re expecting an in-depth explanation of what everything does in this XML file, so you’ll have to Google it. My method for finding what I need is I start clicking around until I find something that resembles the Console. I’ll show you what I mean:

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image4.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb4.png" alt="image" width="625" height="231" border="0" /></a>

Expand NodeDescription –&gt; ChildNodes –&gt; RootNodeDescription. If you click on DisplayName, it shows “OverviewNode”, which is the root node you see in the console. When you expand Overview in the console, you have three child nodes (Apps, Updates, OSD), so expand the ChildNodes node in XML Notepad to see three nodes:

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image5.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb5.png" alt="image" width="1033" height="371" border="0" /></a>

We want Application Management, which is the first node, so expand the first RootNodeDescription.

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image6.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb6.png" alt="image" width="806" height="290" border="0" /></a>

We want the first child node (Applications), so expand “ChildNodes” and then the first “RootNodeDescription” in “ChildNodes”

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image7.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb7.png" alt="image" width="830" height="641" border="0" /></a>

Now, this is the GUID associated with Applications in software library:

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image8.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb8.png" alt="image" width="209" height="237" border="0" /></a>

If I use this GUID, I’m going to be putting a menu on the Applications node in the left pane of the console. I don’t want a menu option there, I want one in the right pane, where all the applications show up. The console uses WQL queries to fill the right pane up with objects, so in the XML document, expand Queries and then QueryDescription:

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image9.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb9.png" alt="image" width="1167" height="572" border="0" /></a>

This is the query used by the console to grab all the application objects. You’ll want to save the NamespaceGuid and the Query for use in parts 2 and 3.

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image10.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb10.png" alt="image" width="901" height="137" border="0" /></a>