---
id: 298
title: 'A ConfigMgr Admin&rsquo;s Guide to WOL Magic Packet: Part One'
date: 2013-12-15T19:05:54+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=298
permalink: /a-configmgr-admins-guide-to-wol-magic-part-one/
categories:
  - ConfigMgr
  - WOL
---
I’m sure everyone reading this has a perfect WOL setup. You just click the button to turn on WOL magic packet on the ConfigMgr server and it starts working, like magic! But, just in case one or two of you don’t have a 100% success rate, I decided to write a guide on things I do to increase your success rate with magic packet.

The first thing you’ll want to do is enable WOL on the ConfigMgr server. Go to the Administration tab, Site Configuration, Sites, right click on the site and go to properties:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-1-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-1_thumb.png" alt="image (1)" width="432" height="305" border="0" /></a>

Go to the Wake On LAN tab and check “Enable Wake On LAN for this site”

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image1" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image1_thumb.png" alt="image1" width="244" height="134" border="0" /></a>

Now, if you don’t have a server with the out of band service point installed in your site, you are going to be greeted by an informational message.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image2" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image2_thumb.png" alt="image2" width="244" height="112" border="0" /></a>

This message has nothing to do with WOL magic packet, it only applies to the AMT power on commands. Magic packets will still be sent out as long as you select option one or option three.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image3" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image3_thumb.png" alt="image3" width="558" height="147" border="0" /></a>

You will have to decide if you want to do subnet directed broadcasts or unicast. Subnet directed broadcasts send the packet to all computers on the last subnet ConfigMgr saw the computer on. Unicast broadcasts send the packet only to that computer. Unicast tends to be not as reliable as subnet directed broadcasts because it depends on the router’s ARP cache. If the cache is cleared, it won’t know where to send the packet. I recommend subnet directed broadcasts. It can be a security problem if you don’t restrict who can send the packets though. So if your network team gives you push back, have them restrict subnet directed broadcasts to your ConfigMgr server.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image4.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image4" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image4_thumb.png" alt="image4" width="244" height="78" border="0" /></a>

Click on Advanced and look through these settings.  The defaults are usually fine, but if you have a special need in your environment, you can usually find the setting to change here. For instance, if you know you have a packet loss problem, you might want to change the number of retries to something more than 3.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image5.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image5" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image5_thumb.png" alt="image5" width="298" height="306" border="0" /></a>

Now, go to the ports tab and check to make sure the WOL port is the one you want. By default it is 9, but you can change it to whatever you want.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image6.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image6" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image6_thumb.png" alt="image6" width="388" height="239" border="0" /></a>