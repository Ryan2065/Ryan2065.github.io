---
id: 222
title: 'Better Know a PowerShell UI: Let&#8217;s go buy a duck (Part 2)'
date: 2016-01-13T12:00:45+00:00
author: Ryan Ephgrave
layout: post
guid: https://www.ephingadmin.com/?p=222
permalink: /better-know-a-powershell-ui-lets-go-buy-a-duck-part-2/
categories:
  - Powershell
  - WPF
tags:
  - ISE
  - Powershell
  - WPF
---
Welcome to the fourth installment of my long running series Better Know a PowerShell UI. Last time we designed an interface in Visual Studio for buying a duck. It’s a very simple user interface because when we buy ducks we want it now and don’t want to have to fill out lots of forms. Today, we are going to move that interface over to PowerShell and make the button buy a duck!

First off, you’ll want to make sure you have the <a href="http://www.ephingadmin.com/run-wpf-powershell-scripts-in-the-ise-without-all-the-crashing/" target="_blank">PowerShell ISE addon</a> I created to open your PowerShell script in a new window.

Open up your duck project and look for the XAML. It should be right under your designer and looks a lot like XML:

<a href="http://ephingadmin.com/wp-content/uploads/2016/01/image-11.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://ephingadmin.com/wp-content/uploads/2016/01/image_thumb-11.png" alt="image" width="644" height="424" border="0" /></a>

Put your curser in the window with the XAML, hit Ctrl+A (select all the text) and then Ctrl+C (copy the text). Paste all of this into a black document in the ISE. Now, let’s make a XAML variable by putting all of this in the [xml]$xaml variable.
<pre class="lang:ps decode:true ">[xml]$xaml = @'
&lt;Window x:Class="Lets_Go_Buy_a_Duck.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Lets_Go_Buy_a_Duck"
        mc:Ignorable="d"
        Title="MainWindow" Height="350" Width="525"&gt;
    &lt;Grid&gt;
        &lt;Button x:Name="DuckButton" Content="Dash!" HorizontalAlignment="Left" VerticalAlignment="Top" Width="75" Click="button_Click" Margin="221,135,0,0"/&gt;
        &lt;Label x:Name="label" Content="You're going to buy a duck!" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="179,104,0,0"/&gt;

    &lt;/Grid&gt;
&lt;/Window&gt;
'@</pre>
To create our UI, we will firstly need to add the WPF assemblies. Here are the assemblies you’ll want to load:
<pre class="lang:ps decode:true ">Add-Type -AssemblyName PresentationFramework,PresentationCore,WindowsBase</pre>
Next, we need to create a XAML window. We need a XMLNodeReader to load into the XAMLReader, so you’ll want these lines of code:
<pre class="lang:ps decode:true ">$XMLReader = (New-Object System.Xml.XmlNodeReader $xaml)
$Window = [Windows.Markup.XamlReader]::Load($XMLReader)</pre>
Hit Alt+F5 (to run it with the extension you installed earlier) and you’ll get this lovely red:

<a href="http://ephingadmin.com/wp-content/uploads/2016/01/image-12.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://ephingadmin.com/wp-content/uploads/2016/01/image_thumb-12.png" alt="image" width="644" height="139" border="0" /></a>

Why are you getting this? Well Visual Studio adds some extra lines of code that are not needed! If you notice, on the first line of your XAML code, it will say something like x:Class=”Lets_Go_Buy_a_Duck.MainWindow”. Remove this because it is telling Visual Studio where in the project this XAML window resides. We don’t have it in Visual studio anymore, so we can remove it. You’ll also want to make a few more edits. Visual Studio 2015 started adding additional namespaces to the XAML. These are at the top and start with xmlns=. By default, you use the x namespace (xmlns:x), and unless you need the rest you can get rid of them. In PowerShell, ‘mc:Ignorable=”d”’ will always error out, so remove the d namespace and the line that says mc:Ignorable=”d”. Once you’re done, your XML should look like this:
<pre class="lang:ps decode:true ">&lt;Window 
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Lets_Go_Buy_a_Duck"
        Title="MainWindow" Height="350" Width="525"&gt;
    &lt;Grid&gt;
        &lt;Button x:Name="DuckButton" Content="Dash!" HorizontalAlignment="Left" VerticalAlignment="Top" Width="75" Margin="221,135,0,0"/&gt;
        &lt;Label x:Name="label" Content="You're going to buy a duck!" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="179,104,0,0"/&gt;

    &lt;/Grid&gt;
&lt;/Window&gt;</pre>
Now, hit Alt+F5 and you should see…. nothing! Well, that means at least there were no errors, right?

Before we go popping up windows, we should make the button do something. We came here with the intention of buying a duck, and we are going to buy a duck! Our button in the XAML is named “DuckButton”, but how do we interact with that? Your $Window variable has a method on it called FindName, which makes it search the XAML for the tool with that name, and it will return the object. So, let’s store that object into $DuckButton like this:
<pre class="lang:ps decode:true ">$DuckButton = $Window.FindName("DuckButton")</pre>
Now that we have a button object, let’s make it do something! Buttons work through things called events. There are a lot of events for everything in WPF. One event that is on almost every control is Click. This will make code run every time the object is clicked on. That sure sounds like what we want! So, let’s make this button open a website with the command Start-Process 'http://www.metzerfarms.com/DucksForSale.cfm'.  That will look like:
<pre class="lang:ps decode:true ">$DuckButton.Add_Click({
    Start-Process 'http://www.metzerfarms.com/DucksForSale.cfm'
})</pre>
Now, if we hit Alt+F5 we get… Nothing! We didn’t add anything to show the window so it’s not going to pop up. Add in the ShowDialog command with this:
<pre class="lang:ps decode:true ">$Window.ShowDialog()</pre>
And now, finally, when you hit Alt+F5 you’ll see your window!

<a href="http://ephingadmin.com/wp-content/uploads/2016/01/image-13.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://ephingadmin.com/wp-content/uploads/2016/01/image_thumb-13.png" alt="image" width="644" height="435" border="0" /></a>

Hit the button and it should take you to a site to buy a duck!

<a href="http://ephingadmin.com/wp-content/uploads/2016/01/image-14.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://ephingadmin.com/wp-content/uploads/2016/01/image_thumb-14.png" alt="image" width="644" height="243" border="0" /></a>

I know what you’re asking yourself, wouldn’t it be easier to simply bookmark this site? Yup.

Join me next time when I teach you how to talk to your new duck with PowerShell!