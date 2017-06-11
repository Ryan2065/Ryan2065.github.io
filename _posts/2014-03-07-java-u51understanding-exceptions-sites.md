---
id: 149
title: Java u51–Understanding Exceptions.Sites
date: 2014-03-07T20:38:42+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=149
permalink: /java-u51understanding-exceptions-sites/
categories:
  - Java
tags:
  - Administer Java
  - Exception.Sites
  - Java Deployment
---
In u45, Java released a ruleset file which was meant to solve system administrator’s problems with whitelisting sites and preventing popups on unsigned sites. The problem with the ruleset was it required administrators to leave their comfort zone and step into the Java developer world by creating jar files and signing them. When I heard the new version of Java came with a new way to whitelist sites, I was pretty excited. I thought Oracle finally included a system administrator friendly way to manage Java… I was wrong.

At first glance, this Exception.Sites file might seem like it has everything admins need. What more do you need to do other than whitelisting sites? Well there are a number of problems you’ll encounter using this method and I’m going to detail them here at the end of the post. If you wish to go ahead and use this method to manage Java (it’s not a horrible way to manage Java, it just isn’t the best way) I will show how you can centrally manage it with SCCM.

Oracle has said the exception.sites file is meant to be for users to manage while the deployment.ruleset is for users to manage. By default, exception.sites lives in the users profile at this path:
C:\Users\UserName\AppData\LocalLow\Sun\Java\Deployment\Security

and each user manages their own site list through the Java control panel:
<p id="EWcfJKu"><img class="alignnone size-full wp-image-150 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_565a104ff2fb3.png" alt="" /></p>
Administrators can change the location of this file for all users through two files in this folder:
C:WindowsSunJavaDeployment

The first file you will want here is a file named deployment.config. If you have a fresh install of Java, you will probably have to create the folder and this file if they don’t exist. This properties file tells Java there are computer settings that need to be set. Any settings you set in the properties file will overwrite the user settings. The content of this file is:
<pre class="lang:default decode:true">deployment.system.config=C\:\\Windows\\Sun\\Java\\Deployment\\deployment.properties
deployment.system.config.mandatory=false</pre>
The second file you need is deployment.properties. This file lists all the properties the admin has set for the computer. By default, there are no settings set so the file will probably not exist. Create it and put this line in:
<pre class="lang:default decode:true ">deployment.user.security.exception.sites=C\:\\Windows\\Sun\\Java\\Deployment\\exception.sites</pre>
This line of code tells Java to use the exception.sites file in C:WindowsSunJavaDeploymentexception.sites instead of the user one. Make sure this file is in a location your users have read access to or this will not work. Also, if you set this and a user has write access to the file, they will be able to change the site list for all users who log into the computer. Java will not use the exception.sites list in their profile anymore.

And that’s it! Now you can manage the exception.sites list.

Why shouldn’t admins use this method?  Well here is a little list:
<ol>
	<li>Oracle provides a better way to manage exceptions through the deploymentruleset</li>
	<li>This method only lets you whitelist. You can not set security levels, block sites, or set which version should be used on which site</li>
	<li>This method doesn’t accept wildcards.</li>
	<li>Users will not be able to add their own exceptions, so all needed exceptions will have to be set by you (this may be a positive, depending on the environment)</li>
</ol>
If you don’t care about these points, then by all means use the Exception.Sites file to manage Java. As I said at the start, it isn’t a horrible way to manage Java, it just doesn’t have all the features of the deploymentruleset.