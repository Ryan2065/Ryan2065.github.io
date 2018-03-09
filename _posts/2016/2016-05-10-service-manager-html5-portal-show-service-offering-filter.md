---
id: 449
title: 'Service Manager HTML5 Portal: Show Service Offering Filter'
date: 2016-05-10T20:46:04+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=449
permalink: /service-manager-html5-portal-show-service-offering-filter/
categories:
  - HTML5 Portal
  - SCSM
tags:
  - HTML5
  - SCSM
  - Service Manager Portal
---
<span style="font-size: small;">I have a lot of Service Offerings in my portal, so I want the filter to be front and center when a user goes to the HTML5 portal. I did a quick edit to the Offerings.cshtml file and was able to make the filter default to showing.</span>

<span style="font-size: small;">Simply go to Portal\Views\Home\Offerings.cshtml and find this line:</span>

 
<pre class="lang:xhtml decode:true " >&lt;input type="search" placeholder="@Resources.SelfServicePortalResources.FindByKeywords" name="searchRequest" id="searchRequest" style="display:none;" oninput="OnSearchOrFilter()"&gt;</pre> 


<span style="font-size: small;">Take out the style to make it default to visible:</span>

 
<pre class="lang:xhtml decode:true " >&lt;input type="search" placeholder="@Resources.SelfServicePortalResources.FindByKeywords" name="searchRequest" id="searchRequest" oninput="OnSearchOrFilter()"&gt;</pre> 


<span style="font-size: small;">Now when someone goes to the portal the filter will be showing!</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/05/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/05/image_thumb.png" alt="image" width="644" height="197" border="0" /></a>