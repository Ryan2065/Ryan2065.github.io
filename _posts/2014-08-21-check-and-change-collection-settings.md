---
id: 129
title: Check and Change Collection Settings
date: 2014-08-21T22:04:36+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=129
permalink: /check-and-change-collection-settings/
categories:
  - ConfigMgr
  - Powershell
tags:
  - Collection
  - Incremental
---
As a consultant, I’m generally going into environments and have no clue how things are set up. For collections, I want to know things like how many collections have incremental updates set up, what are the limiting collections, maintenance windows, power plans, and collection variables.

Believe it or not, I wrote a script to gather this information for me! I also added in the ability to change the limiting collection of the selected collections and / or change the incremental update setting.
<p id="ZIxZvRj"><img class="alignnone size-full wp-image-130 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_56523bca33eb4.png" alt="" /></p>
<a href="https://gallery.technet.microsoft.com/Quickly-Change-Limiting-3a68944a" target="_blank">Download the script here!!</a>

*Note: Really I wrote this tool to change limiting collections and incremental updates, but realized it would take me 30 seconds to add in all the other information, so I did. You can also find most of this information (everything but incremental updates) in the console by adding columns to the collection view.