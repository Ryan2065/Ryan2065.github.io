---
id: 231
title: 'Better Know a PowerShell UI: Quack Like a Duck!'
date: 2016-01-14T09:00:05+00:00
author: Ryan Ephgrave
layout: post
guid: https://www.ephingadmin.com/?p=231
permalink: /better-know-a-powershell-ui-quack-like-a-duck/
categories:
  - Powershell
  - WPF
tags:
  - PoshRSJob
  - Threading
  - WPF
  - XAML
---
If you’ve been following along, you now have a duck and want to interact with it! Since we used PowerShell to buy the duck, let’s use a PowerShell UI to talk to it! You may not know it yet, but you’re about to learn UI threading!

First off, I need to create my user interface. Since I already showed how to do this in previous posts, I’m not going to show it again. Instead I made a video of myself making the user interface and posted that here:

https://youtu.be/F38zHihFyBQ

Now that I have the UI code, I can get started!

I want to set up the UI like I did in the <a href="http://ephingadmin.com/better-know-a-powershell-ui-lets-go-buy-a-duck-part-2" target="_blank">previous post</a> in this series. I’ll add the assemblies, load the Window with the XamlReader, do some magic to set variable names… Wait, what is this?
 
<pre class="lang:ps decode:true " >$xaml.SelectNodes("//*[@Name]") | Foreach-Object { Set-Variable -Name (("Window" + "_" + $_.Name)) -Value $Window.FindName($_.Name) }</pre> 

Recently as I was reading <a href="https://blog.netnerds.net/" target="_blank">Chrissy LeMarie’s blog</a>, I noticed she used this bit of code to find and create all the named controls in XAML code. I stole it, modified it a little, and now use it in most of my UI scripts! This one liner goes through all the xml nodes in your XAML, finds everything with a name, and then creates a variable called $Window_CONTROLNAME for you to use in your script! Very cool.

Anyway, I now have this as my script:
 
<pre class="lang:ps decode:true " >[xml]$xaml = @'
&lt;Window 
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Quack_like_a_duck"
        Title="Quack like a duck" SizeToContent="WidthAndHeight" WindowStartupLocation="CenterScreen" &gt;
    &lt;Grid&gt;
        &lt;Label Content="Hit the button to quack like a duck!" HorizontalAlignment="Center" VerticalAlignment="Top"/&gt;
        &lt;Button Name="Btn_Quack" Content="Quack!" HorizontalAlignment="Center" VerticalAlignment="Top" Width="75" Margin="0,30,0,10"/&gt;
    &lt;/Grid&gt;
&lt;/Window&gt;

'@
# Add assemblies
Add-Type -AssemblyName PresentationFramework,PresentationCore,WindowsBase

# Make window
$Window = [Windows.Markup.XamlReader]::Load((New-Object System.Xml.XmlNodeReader $xaml))
$xaml.SelectNodes("//*[@Name]") | Foreach-Object { Set-Variable -Name (("Window" + "_" + $_.Name)) -Value $Window.FindName($_.Name) }
 
$Window_Btn_Quack.Add_Click({
    
})
 
$null = $Window.ShowDialog()</pre> 

All I have to do is make the button do something and I’m all set! Let’s put in the quacking code! I’m not going to explain how to make your computer talk when <a href="http://learn-powershell.net/2013/12/04/give-powershell-a-voice-using-the-speechsynthesizer-class/" target="_blank">Boe Prox did such a great job</a> doing that. Instead, I’ll take his code and put it in the button!
 
<pre class="lang:ps decode:true " >$Window_Btn_Quack.Add_Click({
    Add-Type -AssemblyName System.speech
    $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
    $speak.Speak("Quack quack quack quack quack")
    Start-Sleep 1
    $speak.Speak("Quack Quack")
    Start-Sleep 1
    $speak.Speak("Quack Quack Quack")
})</pre> 

And I’m done, right?  Well let’s run it and find out!

My computer started quacking, but I think I was over generous with the quacks. My wife gave me a look… Apparently I interrupted her Teen Mom marathon.

I tried to stop it, but the UI was locked up! Why is it locked up? User interfaces run in a thread, and the thread can’t be in use if you want to do anything to the UI (even close it!). How do I fix this?

In order to fix this, I need to run either the UI or the button action in a new thread. Programming best practices say you run the UI in the main thread and put your events (Button presses) in a background thread. So, let’s give that a shot!

To thread this script, I’m going to use <a href="https://github.com/proxb/PoshRSJob" target="_blank">PoshRSJob</a>. This is a module created by Boe Prox, and he has instructions on how to use it <a href="http://learn-powershell.net/2015/03/31/introducing-poshrsjob-as-an-alternative-to-powershell-jobs/" target="_blank">here</a>. With that module, to thread my script I simply need to call Start-RSJob –Name ‘NameThisJob’ –ScriptBlock {Script}. Here’s the code now:
 
<pre class="lang:ps decode:true " >Import-Module PoshRSJob
$Window_Btn_Quack.Add_Click({
    $QuackScript = {
        Add-Type -AssemblyName System.speech
        $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
        $speak.Speak("Quack quack quack quack quack")
        Start-Sleep 1
        $speak.Speak("Quack Quack")
        Start-Sleep 1
        $speak.Speak("Quack Quack Quack")
    }
    Start-RSJob -Name 'QuackJob' -ScriptBlock $QuackScript
})
 </pre> 

Now, run it and listen to the quacks! Your duck will now feel right at home, and you can close it when your significant other gets annoyed!