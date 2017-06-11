---
id: 735
title: 'Graph Week &#8211; Exploring the MSGraphAPI Cmdlets'
date: 2017-01-09T12:00:03+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=735
permalink: /graph-week-exploring-msgraphapi-cmdlets/
categories:
  - Graph
  - Powershell
tags:
  - Classes
  - Graph
  - Microsoft Graph
  - Powershell
---
<span style="font-size: small;"><a href="http://www.ephingadmin.com/graphweekgettingstarted/" target="_blank">Yesterday</a>, I showed you how to install and start with the MSGraphAPI cmdlets. Today we’re going to explore them and see all that you can do with them!</span>

<span style="font-size: small;">We learned how to authenticate yesterday, and run the first command: Get-GraphUsers. What does this command return? I can use GetType to see:</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2017/01/image.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2017/01/image_thumb.png" alt="image" width="644" height="146" border="0" /></a>

<span style="font-size: small;">It’s not just returning a hash or array, it’s returning a custom class called GraphUser_v1. What is this object? This is a custom object created for the cmdlets that has a number of properties and methods on it.</span>

<span style="font-size: small;">What are some of the things you can do with it? Here’s a list of all the Get methods on GraphUser_v1 objects:</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2017/01/image-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2017/01/image_thumb-1.png" alt="image" width="621" height="484" border="0" /></a></p>
<p align="left"><span style="font-size: small;">When I use Get_Manager() on the object, it will query Graph and find the manager of the user object in Azure AD:</span></p>
<p align="left"><a href="http://www.ephingadmin.com/wp-content/uploads/2017/01/image-2.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2017/01/image_thumb-2.png" alt="image" width="644" height="126" border="0" /></a></p>
<p align="left"><span style="font-size: small;">Microsoft Graph is an online service, so different environments may have different properties and methods available to them. Classes in the MSGraphAPI cmdlets are not dynamic, so there may be a time when the Graph cmdlets are returning properties that don’t show up in the property list of users. Because of this, all objects have a property called “RawJSON” to be sure data is never lost. If you don’t see the information you are expecting on the object, check RawJSON! </span></p>
<p align="left"><span style="font-size: small;">Here’s an example of RawJSON on a user object:</span></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2017/01/image-3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2017/01/image_thumb-3.png" alt="image" width="644" height="86" border="0" /></a></p>
<span style="font-size: small;">And that’s it for today! You should now know how to start using the Graph cmdlets and be familiar with the object types that are returned. Tomorrow we will look at querying for new items in Graph using Invoke-GraphMethod!</span>