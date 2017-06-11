---
id: 144
title: Introducing the EphingLab!
date: 2015-11-25T20:34:59+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=144
permalink: /introducing-the-ephinglab/
categories:
  - Powershell
tags:
  - Automated lab
  - PDT
---
I really love and hate the PDT. I love the idea and the work put into it, but hate that I can't easily customize it or really even know what they are doing behind the scenes. Because of this, I decided to make my own lab installer that was completely customizable and easy to use! The lab borrows a lot of code directly from the <a href="https://gallery.technet.microsoft.com/PowerShell-Deployment-f20bb605" target="_blank">PDT</a>, so thank you to Rob Willis for that great tool!

The EphingLab is available on <a href="https://github.com/Ryan2065/EphingLab" target="_blank">GitHub</a> (Open the link and go to the bottom right to download as a zip) and was made to do much less than the PDT, but give you the power to do so much more. My idea was simple, create a domain controller and then let the user do whatever else they wanted. To meet this goal, I have three parts to the script.

The first is the XML lab file. Simply define general settings all VMs will follow (Domain, admin password, switch, and domain controller) and then define the VM settings. One thing that isn't that obvious is &lt;DomainController&gt;Lab-DC&lt;/DomainController&gt; is looking for the name of the VM that is the DC. It uses this information to figure out the DNS address of the other VMs and to know which VM it is going to install the DC role on. Here is sample XML that creates a lab with the VMs Lab-DC and Lab-Test:
<pre class="lang:default decode:true">&lt;Lab&gt;
	&lt;General&gt;
		&lt;Domain&gt;Home.Lab&lt;/Domain&gt;
		&lt;AdministratorPassword&gt;P@ssw0rd&lt;/AdministratorPassword&gt;
		&lt;Switch&gt;Lab-Switch&lt;/Switch&gt;
		&lt;DomainController&gt;Lab-DC&lt;/DomainController&gt;
	&lt;/General&gt;
	&lt;VM&gt;
		&lt;VMName&gt;Lab-DC&lt;/VMName&gt;
		&lt;VHDPath&gt;D:\Hyper-V\Virtual Hard Disks\Lab-DC.VHDX&lt;/VHDPath&gt;
		&lt;VHDParentPath&gt;D:\Hyper-V\Virtual Hard Disks\Server2012R2DataCenter-ParentDisk.vhdx&lt;/VHDParentPath&gt;
		&lt;VMPath&gt;D:\Hyper-V\Virtual Machines&lt;/VMPath&gt;
		&lt;Generation&gt;2&lt;/Generation&gt;
		&lt;Memory&gt;1GB&lt;/Memory&gt;
		&lt;Processors&gt;1&lt;/Processors&gt;
		&lt;ProductKey&gt;&lt;/ProductKey&gt;
		&lt;IPAddress&gt;192.168.1.4&lt;/IPAddress&gt;
		&lt;StartupScript&gt;&lt;/StartupScript&gt;
		&lt;FolderToCopy&gt;&lt;/FolderToCopy&gt;
		&lt;AdditionalDrive&gt;
			&lt;Path&gt;&lt;/Path&gt;
			&lt;Size&gt;&lt;/Size&gt;
		&lt;/AdditionalDrive&gt;
	&lt;/VM&gt;
	&lt;VM&gt;
		&lt;VMName&gt;Lab-Test&lt;/VMName&gt;
		&lt;VHDPath&gt;D:\Hyper-V\Virtual Hard Disks\Lab-Test.VHDX&lt;/VHDPath&gt;
		&lt;VHDParentPath&gt;D:\Hyper-V\Virtual Hard Disks\Server2012R2DataCenter-ParentDisk.vhdx&lt;/VHDParentPath&gt;
		&lt;VMPath&gt;D:\Hyper-V\Virtual Machines&lt;/VMPath&gt;
		&lt;Generation&gt;2&lt;/Generation&gt;
		&lt;Memory&gt;1GB&lt;/Memory&gt;
		&lt;Processors&gt;1&lt;/Processors&gt;
		&lt;ProductKey&gt;&lt;/ProductKey&gt;
		&lt;IPAddress&gt;192.168.1.20&lt;/IPAddress&gt;
		&lt;StartupScript&gt;&lt;/StartupScript&gt;
		&lt;FolderToCopy&gt;&lt;/FolderToCopy&gt;
		&lt;AdditionalDrive&gt;
			&lt;Path&gt;&lt;/Path&gt;
			&lt;Size&gt;&lt;/Size&gt;
		&lt;/AdditionalDrive&gt;
	&lt;/VM&gt;
&lt;/Lab&gt;</pre>
The general settings are:

<strong>Domain</strong> - Domain name. Should be something like lab.home or ryans.awesome
<strong>AdministratorPassword</strong> - Password of the local admin accounts and domain account
<strong>Switch</strong> - Switch name. The switch will be created if it is not already as an internal switch. Create it first if you want the switch to be an external switch
<strong>DomainController</strong> - VM name of the DC. The DC should be listed below.

Each VM should have these settings:

<strong>VMName</strong> - This will be the computer name of the VM and the VM name in Hyper-V
<strong>VHDPath</strong> - Path where the VHD will be stored.
<strong>VHDParentPath</strong> - Right now, I only support differencing disks so the VHD has to be based off a parent. Instructions below for making a parent disk.
<strong>VMPath</strong> - Where should the VM be stored?
<strong>Generation</strong> - What generation of VM is it? Required to be 1 or 2.
<strong>Memory</strong> - Memory in GB of the VM. Right now it only affects the startup memory and past that the memory is dynamic. Will be changed in the future
<strong>Processors</strong> - How many processors should the VM have
<strong>ProductKey</strong> - Windows product key
<strong>IPAddress</strong> - IP Address of the VM. This should be unique. There is no DHCP in the lab right now, so you'll need to specify an IP for all VMs.
<strong>StartupScript</strong> - Path of the script that will run at startup. You don't need to specify anything if you just want a domain joined VM. The script will run as the domain admin account, so it should have any rights you need.
<strong>FolderToCopy</strong> - Any folder specified here will be copied to the root of C. You can use this with the StartupScript property to make your VM do anything you want with Powershell. Multiple folders can be specified with multiple FolderToCopy properties

The second part of this process is the FolderToCopy property in the XML. Any folders here will be copied to the root of C of the VM. The nice thing about this is you can copy all the media for a ConfigMgr lab right to the root of C.

The last part, and the part that makes this so flexible is the StartupScript. This script will run as the domain admin and can install anything you want. There are two sample install scripts in the Install Scripts folder. One for ConfigMgr and the other for SCSM. Neither of these work full yet, but they are there to give you an idea of how to work these. The ConfigMgr script has examples of running commands after the VM is rebooted and the SCSM script has examples of using PSRemoting to set up remote servers and new user accounts.

I still have a bit of work to do to add some more features, but the script as is will create your lab and run the startup scripts. I want to add more memory options, the option to create additional HDs on the VM, and the option to not use a differencing disk.

There is a lot of work to do on the install scripts. I want all System Center components to eventually have install scripts and then other things that might be nice to have, like Exchange.

If you have any automated install scripts or want to do work on this project, feel free to help out on GitHub. You could also leave a comment here if you can't figure out GitHub.