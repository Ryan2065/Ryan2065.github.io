---
id: 721
title: 'Graph Week &#8211; Getting Started with Graph and PowerShell!'
date: 2017-01-08T12:00:15+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=721
permalink: /graphweekgettingstarted/
categories:
  - Graph
  - Powershell
tags:
  - Graph
  - Graph API
  - Microsoft Graph
  - Powershell
---
<span style="font-size: small;">Welcome to Graph week! This week we are going to look at how we can automate common tasks with PowerShell and Microsoft Graph. To help us out, I am working on cmdlets called the <a href="https://www.powershellgallery.com/packages/MSGraphAPI" target="_blank">MSGraphAPI cmdlets</a>. They are in beta now, but far enough along to do anything you need to do in Graph!</span>

<span style="font-size: small;">What is <a href="https://graph.microsoft.io/en-us/" target="_blank">Microsoft Graph</a>? Graph is one endpoint to manage multiple Microsoft APIs. What do I mean by that? In the Graph cmdlets, you can simply run the Get-GraphAuthenticationToken once, and are then able to pull information from Intune, Azure AD, SharePoint, Office 365, and any other service Graph ties into! The purpose of the Graph cmdlets is to let you access this information without having to install and learn multiple cmdlets!</span>
<p style="text-align: center;"><iframe width="480" height="216" src="//giphy.com/embed/11sBLVxNs7v6WA" frameborder="0" webkitallowfullscreen="webkitallowfullscreen" mozallowfullscreen="mozallowfullscreen" allowfullscreen="allowfullscreen"></iframe></p>
<span style="font-size: small;">So, how do you get started with these cmdlets? First, install them from the PowerShell Gallery. If you are on Windows 10 or have PowerShell 5 installed, you can run this command to install them:</span>
<pre class="lang:ps decode:true ">Install-Module -Name MSGraphAPI</pre>
<span style="font-size: small;">Now that you have them installed, you can begin using them! You’ll first need to log into the cmdlets with the Get-GraphAuthenticationToken. You’ll need the credentials of a user who can do what you want to do (imagine that!) and your Azure AD tenant name. Here’s the code I always use to call this function:</span>
<pre class="lang:ps decode:true ">$secpasswd = 'Password' | ConvertTo-SecureString -AsPlainText -Force
$mycreds = New-Object System.Management.Automation.PSCredential ('USERNAME@TENANT.com', $secpasswd)
$GraphParams = @{
    'TenantName' = 'TENANTNAME.onmicrosoft.com'
    'Credential'=$mycreds
}
Get-GraphAuthenticationToken @GraphParams</pre>
<span style="font-size: small;">Now that you’re authenticated, try using the cmdlets! Give Get-GraphUsers a shot and see it spit out a list of all your Azure AD users!</span>

<span style="font-size: small;">And that’s where I’ll leave you for today! Join us tomorrow when we explore the Graph cmdlets more in-depth and take a look at the objects you’re getting back!</span>