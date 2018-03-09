---
id: 701
title: Deploy Nano Server with a SCCM Task Sequence
date: 2016-10-15T14:09:30+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=701
permalink: /deploy-nano-server-sccm-task-sequence/
categories:
  - ConfigMgr
  - OSD
tags:
  - ConfigMgr
  - Nano Server
  - OSD
---
<span style="font-size: small;">One of the cool features that came with Server 2016 is <a href="https://blogs.technet.microsoft.com/windowsserver/2015/04/08/microsoft-announces-nano-server-for-modern-apps-and-cloud/" target="_blank">Nano Server</a>. This is a completely stripped down version of Server 2016 that has no UI components and is designed for speed, agility, and lower resource consumption. I decided to try and see if I could deploy this with SCCM. In my investigation, I found out Nano server does not support the SCCM client, so we can not push updates or manage it.</span>
<p style="text-align: center;"><iframe width="480" height="270" src="//giphy.com/embed/qr7dWYeTpZAWs" frameborder="0" webkitallowfullscreen="webkitallowfullscreen" mozallowfullscreen="mozallowfullscreen" allowfullscreen="allowfullscreen"></iframe></p>
<span style="font-size: small;">So, I’m stuck and can’t continue right? Well technically we don’t need to install the SCCM client to apply a WIM, so that isn’t a deal breaker! I also found an article about <a href="https://blogs.technet.microsoft.com/privatecloud/2016/05/02/nano-server-domain-join-deployment-at-a-scale-part-1-introduction/" target="_blank">how to deploy Nano server with WDS</a>. SCCM uses WDS, so it should be possible! Based on this article, there are three things I need to do to deploy Nano server: add components to the boot image, prepare the Nano Server WIM, and create a new domain join script! </span>
<p align="center"><span style="font-size: small;"><strong>Boot Image</strong></span></p>
<p align="left"><span style="font-size: small;">The first piece of the puzzle is getting our boot image ready. In the WDS article, he runs a script to create a boot image and then modifies some additional files to automatically run a PowerShell script. Since this is a task sequence, we don’t need to do the second part and can just add a step to run the script. So, that leaves us with simply adding components to the boot image. Go to your console and add these components to a <a href="https://developer.microsoft.com/en-us/windows/hardware/windows-assessment-deployment-kit" target="_blank">Windows 10 1607 x64 boot image</a>:</span></p>
<p align="left"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb.png" alt="image" width="644" height="231" border="0" /></a></p>
<span style="font-size: small;">Once that’s done, deploy the boot image to your DPs!</span>
<p align="center"><span style="font-size: small;"><strong>Nano Server WIM</strong></span></p>
<p align="left"><span style="font-size: small;">If you look at the Server 2016 ISO, there’s a folder called NanoServer:</span></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-1.png" alt="image" width="644" height="443" border="0" /></a></p>
<p align="left"><span style="font-size: small;">Inside of it, there’s a file called NanoServer.wim. This is not what you are going to deploy! NanoServer operates much like WinPE boot images where we install components directly into to the WIM. If I want this server to be an IIS server, I need to generate a new Nano Server WIM file with the IIS bits put in. These are all the available packages:</span></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-2.png" alt="image" width="483" height="484" border="0" /></a></p>
<span style="font-size: small;">For the purposes of this example, I’m going to install the IIS components and Storage components because I want to play around with making a Nano Server DP later. Open an elevated command prompt, and run the following command to generate a new Nano Server WIM that will run in a virtual machine:</span>
<pre class="lang:ps decode:true">Import-Module f:\NanoServer\NanoServerImageGenerator\NanoServerImageGenerator.psd1
$NanoParamHash = @{
    Edition='Standard'
    DeploymentType='Guest'
    MediaPath='F:\'
    BasePath='C:\NanoFiles\Base'
    TargetPath='C:\NanoFiles\NanoServerVM\NanoServerVM.WIM'
    Package='Microsoft-NanoServer-IIS-Package'
}
New-NanoServerImage @NanoParamHash -Clustering -Storage -EnableRemoteManagementPort</pre>
<span style="font-size: small;">Once your WIM is generated, add it to SCCM in Operating System Images:</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-3.png" alt="image" width="644" height="406" border="0" /></a></p>
<p align="center"><span style="font-size: small;"><strong>Task Sequence</strong></span></p>
<p align="left"><span style="font-size: small;">Last, but not least, we need to build a task sequence to deploy this image! I created a standard task sequence to deploy an image, and then disabled half the steps:</span></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-4.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-4.png" alt="image" width="375" height="484" border="0" /></a></p>
<span style="font-size: small;">First, I modified the Partition Disk 0 – UEFI step to assign the OSDisk variable:</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-5.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-5.png" alt="image" width="376" height="484" border="0" /></a></p>
<span style="font-size: small;">Then, I changed the Apply Operating System step to use the OSDisk variable:</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-6.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-6.png" alt="image" width="416" height="484" border="0" /></a></p>
<span style="font-size: small;">I didn’t really change the Apply Windows Settings, other than put in my product key:</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-7.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-7.png" alt="image" width="390" height="484" border="0" /></a></p>
<span style="font-size: small;">And lastly, I have a new Domain Join step that runs a <a href="https://gist.github.com/Ryan2065/79838b78643d2311d60cb6147e3b87bf" target="_blank">PowerShell script</a> (download):</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-8.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-8.png" alt="image" width="382" height="484" border="0" /></a></p>
<span style="font-size: small;">The linked PowerShell script is a heavily modified version of the script from the WDS install instructions I referenced at the beginning of the article. It takes the positional parameters MachineName, Domain, User, Password, OSDisk. You can see how I specified them in the above screen shot. Download the script, create a package with the script as content, then reference it in your run command line step above.</span>

<span style="font-size: small;">Lastly, distribute the Task Sequence content, deploy the task sequence, then run it on a VM!</span>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-9.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-9.png" alt="image" width="587" height="484" border="0" /></a></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-10.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-10.png" alt="image" width="644" height="278" border="0" /></a></p>
<p align="center"><a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-11.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-11.png" alt="image" width="644" height="385" border="0" /></a></p>
<span style="font-size: small;">After logging in, we can see it is on the domain!</span>

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/10/image-12.png"><img style="background-image: none; float: none; padding-top: 0px; padding-left: 0px; margin-left: auto; display: block; padding-right: 0px; margin-right: auto; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/10/image_thumb-12.png" alt="image" width="644" height="355" border="0" /></a>

<span style="font-size: small;">And that’s it, we can now deploy Nano Server! I’ll leave it up to you to decide if you want to deploy a server you cannot manage (yet, hopefully)!</span>