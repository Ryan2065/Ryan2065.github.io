---
id: 341
title: 'Customizing the CM12 Console: Part 1 &ndash; AdminUI.ConsoleBuilder.exe'
date: 2013-12-22T19:43:26+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=341
permalink: /customizing-the-cm12-console-part-1/
categories:
  - ConfigMgr
---
I’m apparently a big fan of guides… Anyway, this guide is going to show you how to customize the console by adding and removing items with the ConsoleBuilder.

I don’t think many people realize there is a pretty useful tool that is installed with your CM12 console. The filename is AdminUI.ConsoleBuilder.exe and it is stored in the AdminConsolebin folder. This tool can alter menus in the ConfigMgr console and lets you pick and choose what you see. This can help de-clutter the console for yourself or for your co-workers who might not have the same rights as you.

This part of the guide will show you how to do the easy stuff… removing things! As an example, I’ll walk you through how to remove the “Assign to New Site” options that now appear in R2, for those of us that just have one site. Before you do anything, back up your console. There is no save button in this tool, if you make a change, it is set. Now, is your console folder backed up? Alright, fire up the tool as an admin:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image28.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image28" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image28_thumb.png" alt="image28" width="523" height="106" border="0" /></a>

And then File –&gt; Open –&gt; Connected Console. At the bottom left, you will see the tabs you normally see in the console.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image29.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image29" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image29_thumb.png" alt="image29" width="212" height="244" border="0" /></a>

Click on Assets and Compliance and you will see all the options you can see in the ConfigMgr console.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image30.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image30" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image30_thumb.png" alt="image30" width="237" height="295" border="0" /></a>

I know from my previous annoyances that “Reassign Site” shows up when you right click on a device in Devices.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image31.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image31" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image31_thumb.png" alt="image31" width="343" height="205" border="0" /></a>

Click on Devices in the ConsoleBuilder, and here is where the resemblance to the Console ends, you knew it had to happen sometime:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image32.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image32" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image32_thumb.png" alt="image32" width="454" height="128" border="0" /></a>

Now, where do we go from here? Well, you want to go to the queries tab. The Queries tab will show you what query is ran when someone clicks on the node. If you click on the query, it will also show you all the right click options that are available:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image33.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image33" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image33_thumb.png" alt="image33" width="421" height="136" border="0" /></a>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image34.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image34" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image34_thumb.png" alt="image34" width="430" height="162" border="0" /></a>

Right click options are under “Root Actions”

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image35.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image35" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image35_thumb.png" alt="image35" width="311" height="331" border="0" /></a>

How do we get rid of that pesky Reassign Site?  Just right click on it and select delete!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image36.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image36" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image36_thumb.png" alt="image36" width="242" height="149" border="0" /></a>

You will need to do this for all three queries.

Once that’s done, open up your console (close it and re-open if it was already open) and you will no longer see Reassign Site when you right click on a device!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image37.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image37" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image37_thumb.png" alt="image37" width="236" height="318" border="0" /></a>

And that’s it. See, easy!  Stay tuned for when I show you how to add things to the console, that is a little bit more difficult…