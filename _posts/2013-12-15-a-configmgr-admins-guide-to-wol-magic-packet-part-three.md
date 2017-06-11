---
id: 238
title: 'A ConfigMgr Admin&rsquo;s Guide to WOL Magic Packet: Part Three'
date: 2013-12-15T19:22:12+00:00
author: Ryan Ephgrave
layout: post
guid: https://www.ephingadmin.com/?p=238
permalink: /a-configmgr-admins-guide-to-wol-magic-packet-part-three/
categories:
  - ConfigMgr
  - WOL
tags:
  - BIOS
  - ConfigMgr
  - Magic Packet
  - WOL
---
If you follow Part One and Part Two of this guide, you should have a very high success rate with WOL in your environment. If you don’t, this guide will go through a number of gotchas and troubleshooting steps you can follow to get a better success rate.

Create a test package

I suggest making a collection with your computers that refuse to wake up and a test package to deploy to this collection. For the test package, I suggest making a standard program package and the command line simply should be cmd /c. This way you can deploy the package to these computers as required and not have to worry about changes being made. To re-send packets, you can either delete the deployment and re-create it, or change the deadline of the deployment to a minute into the future.

Maintenance Windows

WOL Packets are sent when a deployment deadline is reached, regardless of maintenance windows. So if you want to use WOL with your deployments, make sure the deadline is set inside your maintenance window or your computers might go back to sleep before the maintenance window hits.

Log Files

If you want to make sure the ConfigMgr server is sending those packets, you will want to check the log files. WolCmgr.log will tell you which clients need to be sent packets and the number of packets sent / retried. Wolmgr.log will tell you when it will wake up the computers.

Are The Packets Sent:

If you really want to be sure the packets are being sent, and want to make sure they are going to the right MAC address, use a WOL sniffer on the server. The wol packet sniffer I use is by ProfShutdown and you <a href="http://www.profshutdown.com/download.aspx">can get it here (last link on page).</a>  Run this program as admin, and you will see this screen:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image26.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image26" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image26_thumb.png" alt="image26" width="390" height="304" border="0" /></a>

Whenever a magic packet is sent, you will see all the information on the packet in this screen:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image27.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image27" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image27_thumb.png" alt="image27" width="377" height="292" border="0" /></a>

If you know which computer isn’t waking up, you can copy and paste this information into notepad and do a ctrl+f for that MAC address. If the MAC address is listed, you know the packet is being sent. If it isn’t listed, make sure there is a Hardware Inventory on that computer with the correct MAC address.

Are The Packets Received:

Sometimes the packets are not being received by the computer, and this is the reason why it isn’t waking up. Go to the computer with the problem, turn it on, and log in. Open the WOL Sniffer program I had you download earlier as an admin, and then send packets to that computer. If you see the packet on the computer, you know the packets can get to the computer, so you can generally rule out a network problem. If you are using Unicast, remember the packet won’t be sent to the computer if the router’s ARP cache is cleared. These are generally cleared on a schedule, and the default is usually 30 minutes. Talk to your networking guys about increasing the time between cache clears, or switch to subnet directed broadcasts.

Is the NIC On:

If there are no link lights on the back of the computer, where the network cord is plugged in, then your NIC is probably not getting any power. Because of this, it won’t be able to turn on the computer when packets are sent. Check BIOS and / or Windows to see which is turning off the NIC. The next trick will tell you how to narrow down which is causing the problem.

Is This A Windows Problem or BIOS Problem:

One interesting trick you can do is power off the computer, and then power it back on going into BIOS. Sometimes, Windows turns the NIC off when the computer is powered down, and this action will turn back on the NIC. Now, power off the computer (make sure it doesn’t boot into Windows) and try to wake up the computer. If it wakes up, and it never woke up before, you are probably looking at a Windows setting problem or a driver bug. If it doesn’t wake up, check the BIOS. make sure all the settings are correct, and if they are look into updating or downgrading the BIOS.

If you have any other troubleshooting tips, please let me know in the comments!