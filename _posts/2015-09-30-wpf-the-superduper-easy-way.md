---
id: 29
title: WPF The SuperDuper Easy Way!
date: 2015-09-30T22:33:29+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=29
permalink: /wpf-the-superduper-easy-way/
categories:
  - Powershell
  - WPF
tags:
  - Powershell
  - User Interface
  - WPF
  - XAML
---
I’ve posted a number of times about WPF and how easy it is, but I decided to make it even easier! I wrote a function to write all the additional code for me to make my WPF window pop up. All I have to do is give it valid XAML code and let it do it’s magic!

Simply design a window in Visual Studio (the free version is fully featured to design WPF) and then copy and paste the XAML into the ISE as the variable $xaml. Here’s my sample XAML:
<pre class="lang:ps decode:true">[xml]$xaml = @'
&lt;Window
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="350" Width="525"&gt;
    &lt;Grid&gt;
        &lt;Button Name="Button" Content="Button" Width="100" Height="100"/&gt;
        &lt;Label Name="Label" Content="This is my label!" HorizontalAlignment="Left" VerticalAlignment="Top"/&gt;
 
    &lt;/Grid&gt;
&lt;/Window&gt;
 
'@
</pre>
Remember to remove the x:Class=”” bit from the first line. This is there to tell the compiler where the code behind the window is. In Powershell, we don’t have any code behind the XAML so this will cause the Window to error out. <a href="https://msdn.microsoft.com/en-us/library/cc189082%28v=vs.95%29.aspx" target="_blank">You can read more about x:Class here if you want to learn more!</a>

Now that we have our XAML in a fresh Powershell window, run it! Why? The XAML variable needs to be loaded into the Powershell session for this function to work, so it needs to be run. If you have other code in the script you don’t want to load, highlight the XAML code and hit F8:
<p id="jiZkyex"><img class="alignnone wp-image-49 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bafe76ea52.png" alt="" width="1086" height="245" /></p>
Now, simply run the function New-EphingWPFCode and it will create the variable $Window and write all the code you need to start! It will also add in the click events if it finds any Button or MenuItem controls.
<pre class="lang:ps decode:true ">New-EphingWPFCode -xaml $xaml</pre>
<p id="nVrwtvM"><img class="alignnone size-full wp-image-52 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564bb00e1e62f.png" alt="" /></p>
Hit Run and watch your new Window pop up! Add any code into the click event of the button and it will run when the button is clicked. The variables created are all $Window_NAME where Name is what they were called in the XAML. If you already have a Window variable and want a different name, you can specify the -VariableName switch to change it.

Let me know if you have any issues! Also, make sure you check out <a href="http://ephingadmin.com/run-wpf-powershell-scripts-ise-without-crashing/" target="_blank">my run in a new window ISE extension</a> to run scripts in a new Powershell window. WPF tends to make the ISE crash if you don’t do this.

Scroll to the bottom and click "View Raw" to get a nice copy/paste version of the code!

https://gist.github.com/Ryan2065/c5bf40ac7cf5f1b7860a