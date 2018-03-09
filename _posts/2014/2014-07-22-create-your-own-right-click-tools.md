---
id: 112
title: Create Your Own Right Click Tools
date: 2014-07-22T21:55:52+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=112
permalink: /create-your-own-right-click-tools/
categories:
  - ConfigMgr
  - Right Click Tools
tags:
  - Console
  - Extensions
  - Right click tools
  - System Center Configuration Manager
---
I decided to write an updated version of this blog post since the old one used a tool I no longer use (XML Notepad) and now use Notepad++ for my right click tools writing. I hope you enjoy this guide!
<p align="center">Step 1: Find the GUID</p>
Each right click action in the Configuration Manager console has a GUID associated with it. Using this GUID we can add actions to the console, but you first need to find it. I wrote a script months ago to help me find the GUID associated with any right click menu.

**Make sure you back up your actions folder before you run this script. You can back it up by going to your AdminConsole install folder \XmlStorage\Extensions and rename the Actions folder to Actions.old. If you installed the console to the default location, this folder is in:
<pre class="lang:ps decode:true">C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\XmlStorage\Extensions</pre>
If you do not have an Extensions folder or an Actions folder, don’t worry about this!

Here is the script:
<pre class="lang:ps decode:true">$ConfigInstallPath = $env:SMS_ADMIN_UI_PATH | Out-String
$ConfigInstallPath = $ConfigInstallPath.Trim()
$XMLPath = $ConfigInstallPath -replace "\\bin\\i386", "\XmlStorage\ConsoleRoot"
$ActionsPath = $ConfigInstallPath -replace "\\bin\\i386", "\XmlStorage\Extensions\Actions"
Get-ChildItem "$XMLPath" -Filter "*.xml" -Recurse | ForEach-Object {
	$FullFileName = $_.FullName
	$FileContent = Get-Content $FullFileName
	foreach ($line in $FileContent) {
		if ($line.ToUpper().Contains("NAMESPACEGUID=")) {
			$SplitLine = $line.Split("`"")
			$GUID = $SplitLine[1]
 
			$FilePath = "$ActionsPath\$GUID"
			New-Item -ItemType Directory -Path $FilePath -ErrorAction SilentlyContinue | Out-Null
			$strOutput = "&lt;ActionDescription Class=`"Executable`" DisplayName=`"$GUID`" MnemonicDisplayName=`"$GUID`" Description=`"$GUID`"&gt;`n"
			$strOutput = $strOutput + "&lt;ShowOn&gt;&lt;string&gt;ContextMenu&lt;/string&gt;&lt;/ShowOn&gt;`n"
			$strOutput = $strOutput + "&lt;Executable&gt;&lt;FilePath&gt;cmd.exe&lt;/FilePath&gt;`n"
			$strOutput = $strOutput + "&lt;Parameters&gt; /c Powershell.exe Add-Type -AssemblyName 'System.Windows.Forms';[Windows.Forms.Clipboard]::SetText('$GUID')&lt;/Parameters&gt;&lt;/Executable&gt;`n"
			$strOutput = $strOutput + "&lt;/ActionDescription&gt;"
			$strOutput &gt; "$FilePath\File.xml"
		}
	}
}</pre>
You’ll want to run the script as an administrator so it can add files in C:\Program Files, or wherever your console is installed.

Once it is run, it creates a right click menu everywhere in the console, even in spots that don’t currently have menus. To get the menus, close and re-open your console.
<p id="tKlqHwX"><img class="alignnone size-full wp-image-113 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_565236921d119.png" alt="" /></p>
I created a new Software Update Group tool and need to add a menu there. The idea behind the tool is to right click on a Software Update Group, and it will let you look at all the Superseded/Expired and allow you to remove those updates from the group. To get the GUID I’ll need, I right click on any of my Software Update Groups. The GUID will be the same for all of them:
<p id="OfzfolI"><img class="alignnone size-full wp-image-114 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_565236b4315c1.png" alt="" /></p>
Click on the GUID (the thing that starts with 2853) and the GUID is now copied to your clipboard! Open Notepad and paste it into there to save for later. If it is not copied to your clipboard, well something in my script didn’t work and you’ll have to write it down the old fashioned way. Now, go back to your Extensions folder, delete the Actions folder, and rename Actions.old to Actions. Once again, the Extensions folder will be here if the Console was installed with default paths:
<pre class="lang:ps decode:true">C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\XmlStorage\Extensions</pre>
Go into your actions folder, create a new folder, and name it the GUID. In this example, I’ll create a folder and name it 2853886b-cce5-4ed4-af43-df69efb2e7d8  If the Console was installed with default paths, the new folder will be:
<pre class="lang:ps decode:true">C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\XmlStorage\Extensions\Actions\2853886b-cce5-4ed4-af43-df69efb2e7d8</pre>
<p align="center">Step 2: The XML</p>
You’ll want to create a XML file in this folder and it can be named whatever you want. The order of the menus in the console is determined alphabetically by filename. I use Notepad++ to edit xml files, so open Notepad++ as an admin and open your new XML file. XML is fairly straightforward once you are used to it, and lucky for us we don’t need to know much for this! If you are completely new to XML and get lost in these examples, <a href="http://www.xmlfiles.com/xml/xml_syntax.asp">this guide on elements</a> and <a href="http://www.xmlfiles.com/xml/xml_attributes.asp">this guide on attributes</a> should catch you up.

Now that you are caught up, we can start writing the XML. Each item on a right click menu, even sub menus, are the element ActionDescription. There are two types of ActionDescripion that I use, Group and Executable. Group will make a submenu while executable will allow you to call a command line.

First, I’ll show you how to make an executable.

When I create a new element, I always close it off right away to avoid confusion later. The first element we need to make (the root element) is ActionDescription:
<pre class="lang:ps decode:true">&lt;ActionDescription&gt;
 
&lt;/ActionDescription&gt;</pre>
ActionDescription will need some attributes to tell the console what it is. You’ll want the Class description to tell it what type this is (Executable), and the DisplayName attribute to show what the title should be. You can also add a Description attribute if you want to add a note of what this does, but it isn’t needed:
<pre class="lang:ps decode:true ">&lt;ActionDescription Class ="Executable" DisplayName="Remove Expired Updates" MnemonicDisplayName="Test" Description="Not a required value"&gt;
 
&lt;/ActionDescription&gt;</pre>
Next, you need a sub element to tell the console where this menu should go. The sub element is called ShowOn and the two values I use are ContextMenu (Right Click Menu) and DefaultHomeTab (Ribbon).
<pre class="lang:ps decode:true ">&lt;ActionDescription Class ="Executable" DisplayName="Remove Expired Updates" MnemonicDisplayName="Test" Description="Not a required value"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
 
&lt;/ActionDescription&gt;</pre>
Since this is an executable, you’ll need the Executable element:
<pre class="lang:ps decode:true ">&lt;ActionDescription Class ="Executable" DisplayName="Remove Expired Updates" MnemonicDisplayName="Test" Description="Not a required value"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
	&lt;Executable&gt;
 
	&lt;/Executable&gt;
&lt;/ActionDescription&gt;</pre>
Next, you need a FilePath element to tell the console what program will be called. You can also specify some parameters for the file with the Parameters element. I am calling a vbs script, so I need to specify wscript.exe as the executable and the vbs script as the parameter:
<pre class="lang:ps decode:true ">&lt;ActionDescription Class ="Executable" DisplayName="Remove Expired Updates" MnemonicDisplayName="Test" Description="Not a required value"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
	&lt;Executable&gt;
		&lt;FilePath&gt;"wscript.exe"&lt;/FilePath&gt;
		&lt;Parameters&gt; "C:\SilentOpenPS.vbs"&lt;/Parameters&gt;
	&lt;/Executable&gt;
&lt;/ActionDescription&gt;</pre>
And now, you’re done! Open the console and see your new menu:
<p id="ZwnrHbm"><img class="alignnone size-full wp-image-115 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523824d160d.png" alt="" /></p>
Now, I’ll show you how to make a submenu. In Notepad++, create the ActionDescription element.
<pre class="lang:ps decode:true">&lt;ActionDescription&gt;
 
&lt;/ActionDescription&gt;</pre>
You’ll need some attributes. The ones I use are DisplayName, Class, and Description. We are making a submenu, so the Class is Group. DisplayName is whatever you want the menu to be in the console, and Description isn’t required, but is nice to say what it is:
<pre class="lang:ps decode:true ">&lt;ActionDescription Class="Group" DisplayName="Software Update Group Actions" Description="The main Software Update group"&gt;
 
&lt;/ActionDescription&gt;</pre>
Next, you need a sub element to tell the console where this menu should go. The sub element is called ShowOn and the two values I use are ContextMenu (Right Click Menu) and DefaultHomeTab (Ribbon).
<pre class="lang:ps decode:true ">&lt;ActionDescription Class="Group" DisplayName="Software Update Group Actions" Description="The main Software Update group"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
	
&lt;/ActionDescription&gt;</pre>
Now, we need the element ActionGroups since this is a sub menu:
<pre class="lang:ps decode:true ">&lt;ActionDescription Class="Group" DisplayName="Software Update Group Actions" Description="The main Software Update group"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
	&lt;ActionGroups&gt;
		
	&lt;/ActionGroups&gt;
&lt;/ActionDescription&gt;</pre>
And now we can put the executable element here. You can just copy and paste the executable element from before in the ActionGroups, and fix the spacing so it looks nice:
<pre class="lang:ps decode:true ">&lt;ActionDescription Class="Group" DisplayName="Software Update Group Actions" Description="The main Software Update group"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
	&lt;ActionGroups&gt;
		&lt;ActionDescription Class ="Executable" DisplayName="Remove Expired Updates" MnemonicDisplayName="Test" Description="Not a required value"&gt;
			&lt;ShowOn&gt;
				&lt;string&gt;ContextMenu&lt;/string&gt;
				&lt;string&gt;DefaultHomeTab&lt;/string&gt;
			&lt;/ShowOn&gt;
			&lt;Executable&gt;
				&lt;FilePath&gt;"wscript.exe"&lt;/FilePath&gt;
				&lt;Parameters&gt; "C:\SilentOpenPS.vbs" &lt;/Parameters&gt;
			&lt;/Executable&gt;
		&lt;/ActionDescription&gt;
	&lt;/ActionGroups&gt;
&lt;/ActionDescription&gt;</pre>
Save, close and re-open your console and you now have a sub menu:
<p id="cqxKEHa"><img class="alignnone size-full wp-image-116 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_565239031842f.png" alt="" /></p>
<p style="text-align: center;">Step 3: Find the Parameters</p>
So, are we done now? Nope!  Now you need to know how to add in Configuration Manager specific parameters. So, the whole purpose of the tool I created was to show you information about the Software Update Group I clicked on. How does my script know which group I selected? The configuration manager console displays information through WQL queries. If we know the WQL query the console used to display information to us, we can use that to send this information to our script.

Hopefully you saved that GUID from the beginning. If not, this is probably your first time doing this and you can use the practice anyway. Go back and get the GUID. I’m adding a menu to Software Update Groups so the GUID I’m using is 2853886b-cce5-4ed4-af43-df69efb2e7d8. Go to the ConsoleRoot folder of your Configuration Manager Console located here if you installed with default paths:
C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\XmlStorage\ConsoleRoot
and Right Click + Edit the SoftwareLibraryNode.xml file. I’m opening this file because I’m on the Software Library Node in the console:
<p id="QnkvqIx"><img class="alignnone size-full wp-image-117 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_5652394d9c736.png" alt="" /></p>
Here is a list of the nodes and what XML file drives them:

Assets and Compliance – AssetManagementNode.xml
Software Library – SoftwareLibraryNode.xml
Monitoring – MonitoringNode.xml
Administration – SiteConfigurationNode.xml
Tools – ToolsNode.xml

Once you are in the XML file for your node, do a Ctrl+F for the GUID. The line below the GUID should be the query used. For Software Update Groups, the query used to display information is:
<p id="ZMdgifx"><img class="alignnone size-full wp-image-118 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_5652397228f4d.png" alt="" /></p>
Select * From SMS_AuthorizationList ORDER BY LocalizedDisplayName

See, it is pulling all information from SMS_AuthorizationList so that means we can pass any* property in the class SMS_AuthorizationList  (I have not been able to pass arrays as arguments yet, so not all types of properties can be passed). Google SMS_AuthorizationList for a list of properties we can access. When you google SMS_AuthorizationList, the first result is for SCCM 2007. Make sure you find the MSDN article for 2012. In this case, I had to google “SMS_AuthorizationList 2012” to find what I needed.

My script requires four bits of information. The name of the group, the ID of the group, the server name of the CM server, and the namespace where the WQL information is stored. Luckily, you can pass all these things to the script! For the name, you want LocalizedDisplayName. This is the name used for a number of things in ConfigMgr, so if you can’t find the name of something, look under this property. For ID, we want CI_ID. I’m not sure what CI stands for, but I’ve noticed a number of IDs are stored in this field. Server is __Server and Namespace is __Namespace. Server and Namespace will always be there no matter what the MSDN article says. You should always be able to use those fields.

To add these to our XML file, we need to pass the parameters in a specific syntax. Here is the syntax: “##SUB:[Parameter]##”. Here is my example XML file with those parameters put in:
<pre class="lang:ps decode:true">&lt;ActionDescription Class="Group" DisplayName="Software Update Group Actions" Description="The main Software Update group"&gt;
	&lt;ShowOn&gt;
		&lt;string&gt;ContextMenu&lt;/string&gt;
		&lt;string&gt;DefaultHomeTab&lt;/string&gt;
	&lt;/ShowOn&gt;
	&lt;ActionGroups&gt;
		&lt;ActionDescription Class ="Executable" DisplayName="Remove Expired Updates" MnemonicDisplayName="Test" Description="Not a required value"&gt;
			&lt;ShowOn&gt;
				&lt;string&gt;ContextMenu&lt;/string&gt;
				&lt;string&gt;DefaultHomeTab&lt;/string&gt;
			&lt;/ShowOn&gt;
			&lt;Executable&gt;
				&lt;FilePath&gt;"wscript.exe"&lt;/FilePath&gt;
				&lt;Parameters&gt; "C:\SilentOpenPS.vbs" "##SUB:LocalizedDisplayName##" "##SUB:CI_ID##" "##SUB:__Namespace##" "##SUB:__Server##"&lt;/Parameters&gt;
			&lt;/Executable&gt;
		&lt;/ActionDescription&gt;
	&lt;/ActionGroups&gt;
&lt;/ActionDescription&gt;</pre>
Now, those parameters will be passed to my script and it will know what Software Update Group I’m clicking on!