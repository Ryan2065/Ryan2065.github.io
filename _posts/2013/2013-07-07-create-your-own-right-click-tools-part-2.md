---
id: 12
title: Create Your Own Right Click Tools – Part 2
date: 2013-07-07T02:01:29+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=12
permalink: /create-your-own-right-click-tools-part-2/
categories:
  - Right Click Tools
---
In <a href="http://ephingadmin.com/wp/create-your-own-right-click-tools-part-1/" target="_blank">part one</a> of this series, I walked you through the steps I follow to find the GUID of a menu in ConfigMgr 2012. We found the GUID of the “Application” node in the Software Library tab is: 968164ab-af86-459c-b89e-d3a49c05d367. Now, I’ll show you how to use that GUID to make a right click menu option in the console.

When the ConfigMgr 2012 console is started up, it uses the XML files we just looked at to build the console. Those XML files handle everything, from creating menus, to the queries needed to display information.  After those XML files are processed, it looks to the Extensions folder for any additional, user created menus. In a default installation of the ConfigMgr 2012 console, there is no extensions folder, so we need to create it.  Navigate to:

C:Program Files (x86)Microsoft Configuration ManagerAdminConsoleXmlStorage
<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/07/image.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/07/image_thumb.png" alt="image" width="1024" height="339" border="0" /></a>

and create a folder called Extensions. Now create a new folder in the Extensions folder called Actions. This is where the right click tool XML files live.  Each menu needs it’s own GUID folder. To create our Application right click menu, create a folder with the GUID’s name.  When all is said and done, you should have this folder structure:

C:Program Files (x86)Microsoft Configuration ManagerAdminConsoleXmlStorageExtensionsActionsf5eb6c20-3c21-4f9d-aeed-d54d39530ff9
<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image11.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb11.png" alt="image" width="889" height="161" border="0" /></a>

Now that we have our folder structure down, it’s time to make the XML file for this menu. First off, what should you name the XML file?  Well that doesn’t matter. The console processes them in alphabetical order, so files higher up will appear higher up on the menu, but really you can name it whatever you want as long as it is a XML file. Create a new text document called “Content Status.xml” in this folder.

<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/07/image2.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/07/image_thumb2.png" alt="image" width="778" height="190" border="0" /></a>

Now here is another part of the guide where I can’t tell you “why”, I can just tell you what to do.  Open your Content Status.xml file in Notepad (NOT XML Notepad).  We need to create a menu item that will execute a powershell script. Here is the syntax for that:

&lt;ActionDescription Class=”Executable” DisplayName=”Content Status”&gt;
&lt;ShowOn&gt;&lt;string&gt;ContextMenu&lt;/string&gt;&lt;/ShowOn&gt;
&lt;Executable&gt;
&lt;FilePath&gt;Powershell.exe&lt;/FilePath&gt;
&lt;Parameters&gt;-executionpolicy bypass -file “C:Content Status.ps1″&lt;/Parameters&gt;
&lt;/Executable&gt;
&lt;/ActionDescription&gt;

The first line, we give the name and set the class. I want this to execution a powershell script, so I set it as Executable. The second line determines what our menu item will show on. Right click menus are called Context Menus. Line three starts the Executable node, which has the file path and parameters options. Line 6 and 7 close the Executable and ActionDescription nodes.

Save your XML document, and open up the console.  Navigate to Applications, and right click on any application.  Your new menu should now be there at the bottom of the list!
<a href="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image12.png"><img title="image" src="http://ephingadmin.azurewebsites.net/wp-content/uploads/2013/08/image_thumb12.png" alt="image" width="670" height="127" border="0" /></a>
In Part 3, I’ll go through some tips and tricks I’ve found as I’ve made the Powershell Right Click Tools, including how to pass variables to the scripts.