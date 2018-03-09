---
id: 70
title: Safe Ribbon Names
date: 2015-04-16T02:17:55+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=70
permalink: /safe-ribbon-names/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Collection
  - Console
---
One of the major issues with the Configuration Manager console is the ribbon doesn’t always tell you what thing you are performing an action on. For instance, if I open up a device collection and then click on a device, the ribbon still stays on Collection, so any action I do from the ribbon (like, adding it to an Imaging collection) will be performed on all devices in the collection instead of the device I selected:

&nbsp;
<p id="TjnTGUq"><img class="alignnone  wp-image-71 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bdf53bdb6b.png" alt="" width="552" height="291" /></p>
As you can see, the ribbon options look like they are for devices, but it is still on the Collection tab. If you aren’t paying close attention and select “Add Selected Items”, you will be imaging All Systems instead of the VM I have selected. I was talking with Nash Pherson, and he said he wished the Product Team just re-named the option Add Selected Devices or Add Selected Collections, based on what you are on. I said he didn’t have to wait for the product team, just change the names in the XML! Since I knew Nash wouldn’t accept anything but an automated solution to do this, I wrote a script to change the names.

<a href="https://gallery.technet.microsoft.com/Configuration-Manager-Safe-60ee9b93" target="_blank">You can download the script here!</a>

Run the script and it will change those names to safer ones:
<p id="WrzaSNM"><img class="alignnone size-full wp-image-72 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bdfb6d4ed2.png" alt="" /></p>