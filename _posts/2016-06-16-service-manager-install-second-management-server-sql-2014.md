---
id: 479
title: 'Service Manager: Install Second Management Server with SQL 2014'
date: 2016-06-16T14:47:06+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=479
permalink: /service-manager-install-second-management-server-sql-2014/
categories:
  - "Reasons I'm Going Bald"
  - SCSM
tags:
  - SCSM
  - Secondary Management Server
  - Service Manager
  - SQL 2014
---
<span style="font-size: small;">I’ve been working with Service Manager the past few months and have run into some weird stuff, but this past problem is the worst so far.</span>

<span style="font-size: small;">If you <a href="https://technet.microsoft.com/en-us/library/dn281933(v=sc.12).aspx" target="_blank">look at the documentation</a>, SQL 2014 is supported for SCSM 2012 R2 in all scenarios except:</span>

<em>SQL Server 2014 is only supported as a database server for installations of Data Protection Manager (DPM), and Service Manager with Update Rollup 6 applied</em>

<span style="font-size: small;">I’m running 2012 R2 UR9, so I’m good right?</span>

<span style="font-size: small;"><iframe width="480" height="263" src="//giphy.com/embed/f9QdKbhBdzyYo" frameborder="0" webkitallowfullscreen="webkitallowfullscreen" mozallowfullscreen="mozallowfullscreen" allowfullscreen="allowfullscreen"></iframe>
</span>

<span style="font-size: small;">Everything was running well until I wanted to install a new management server a week later. You see, the install files for a new management server are 2012 R2 UR0, which don’t work on SQL 2014. If you try to install with the UI, it will error out and say no SQL 2012 installation was found. I started Googling around, and found <a href="https://blogs.technet.microsoft.com/servicemanager/2015/07/30/deploying-secondary-management-server-after-sql-2014-upgrade/" target="_blank">this blog post</a> about installing with the command line. I see that I can install a secondary management server, but only if I install a dummy SQL 2012 instance first on the SQL 2014 server.</span>

<span style="font-size: small;"><iframe width="480" height="334" src="//giphy.com/embed/124RPAgUcdv5U4" frameborder="0" webkitallowfullscreen="webkitallowfullscreen" mozallowfullscreen="mozallowfullscreen" allowfullscreen="allowfullscreen"></iframe>
</span>

<span style="font-size: small;">Ok, fine, I’ll do it. I install the dummy SQL instance and run the command line:</span>

<em>setup /Silent /Install:Server /AcceptEula <strong>/UseExistingDatabase:SERVERNAME:ServiceManager</strong> /AdminRoleGroup:"DOMAIN\AD Group" /ManagementGroupName:MGName /ServiceRunUnderAccount:Domain\WA\Password /WorkflowAccount:Domain\WA\Password /EnableErrorReporting:No /CustomerExperienceImprovementProgram:No /ProductKey:"KEY"</em>

<span style="font-size: small;">I’m not specifying an instance name because I used the named instance. That threw the installer off. It looked for an instance called SERVERNAME on localhost, it didn’t use the remote SQL server with the default instance. Let’s try that again:</span>

<em>setup /Silent /Install:Server /AcceptEula <strong>/UseExistingDatabase:SERVERNAME\:ServiceManager</strong> /AdminRoleGroup:"DOMAIN\AD Group" /ManagementGroupName:MGName /ServiceRunUnderAccount:Domain\WA\Password /WorkflowAccount:Domain\WA\Password /EnableErrorReporting:No /CustomerExperienceImprovementProgram:No /ProductKey:"KEY"</em>

<span style="font-size: small;">Now, I get a different error message!</span>
<pre class="lang:default decode:true ">05:20:54:Getting SQL default Data Path
05:20:54:***ERROR*** StartService: System.InvalidOperationException: Service MSSQL$ was not found on computer 'SQLSERVERNAME'. ---&gt; System.ComponentModel.Win32Exception: The specified service does not exist as an installed service
   --- End of inner exception stack trace ---
   at System.ServiceProcess.ServiceController.GenerateNames()
   at System.ServiceProcess.ServiceController.get_ServiceName()
   at System.ServiceProcess.ServiceController.GenerateStatus()
   at System.ServiceProcess.ServiceController.get_Status()
   at Microsoft.SystemCenter.Essentials.SetupFramework.HelperClasses.SetupValidationHelpers.StartService(String ServiceName, String ComputerName, Boolean Restart)
05:20:54:Connecting to Remote SQL server SQLSERVERNAME
05:20:54:***ERROR*** Validate Arguments: A required SQL Server service is not running on SQLSERVERNAME: MSSQL$</pre>
<span style="font-size: small;">So, it’s looking for a service on the remote server called MSSQL$, but it doesn’t exist. I agree, that’s not a valid service and does not exist. I verified that service did not exist, and even asked Jes Borland (a SQL MVP coworker) about it. She said that’s not a SQL service.</span>

<span style="font-size: small;"><iframe width="480" height="204" src="//giphy.com/embed/qpFg2k3Njuq2Y" frameborder="0" webkitallowfullscreen="webkitallowfullscreen" mozallowfullscreen="mozallowfullscreen" allowfullscreen="allowfullscreen"></iframe>
</span>

<span style="font-size: small;">Well, let’s try specifying the named instance! Alright, new command line!</span>

<em>setup /Silent /Install:Server /AcceptEula <strong>/UseExistingDatabase:SERVERNAME\MSSQLSERVER:ServiceManager</strong> /AdminRoleGroup:"DOMAIN\AD Group" /ManagementGroupName:MGName /ServiceRunUnderAccount:Domain\WA\Password /WorkflowAccount:Domain\WA\Password /EnableErrorReporting:No /CustomerExperienceImprovementProgram:No /ProductKey:"KEY"</em>

<span style="font-size: small;">I get the same error as above, but now it says the service MSSQL$MSSQLSERVER service doesn't exist. Oh boy…</span>

<span style="font-size: small;">I checked with Steve Buchanan (System Center MVP) and he suggested using a SQL alias. So, I tried that! <a href="https://www.mssqltips.com/sqlservertip/1620/how-to-setup-and-use-a-sql-server-alias/" target="_blank">Here’s a good article on how to do it</a>. I happened to have the SQL Server Configuration Manager tool installed on the system, so I used that. According to the article, there’s a client utility cliconfig.exe that can be used instead of installing the SQL management tools. This only sets it for x64 processes, so be aware of that! You may need the full tools to set x32 and x64. Also, you are setting the SQL alias on the secondary management server. This should not be set on the SQL server! Here’s how I set the alias:</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/06/image.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/06/image_thumb.png" alt="image" width="431" height="484" border="0" /></a>

<span style="font-size: small;">This is saying, if something looks for the SQL instance SERVERNAME on localhost, redirect them to the default instance on SERVERNAME. Now that this is set on my secondary management server, I can try the first command line again:</span>

setup /Silent /Install:Server /AcceptEula /UseExistingDatabase:SERVERNAME:ServiceManager /AdminRoleGroup:"DOMAIN\AD Group" /ManagementGroupName:MGName /ServiceRunUnderAccount:Domain\WA\Password /WorkflowAccount:Domain\WA\Password /EnableErrorReporting:No /CustomerExperienceImprovementProgram:No /ProductKey:"KEY"

<span style="font-size: small;">Eight hours and two name-drops later, it successfully installed!</span>

<span style="font-size: small;"><iframe width="480" height="256" src="//giphy.com/embed/3o6gE6muVWPM2OyOsw" frameborder="0" webkitallowfullscreen="webkitallowfullscreen" mozallowfullscreen="mozallowfullscreen" allowfullscreen="allowfullscreen"></iframe>
</span>