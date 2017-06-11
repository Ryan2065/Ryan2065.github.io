---
id: 404
title: 'Better Know a PowerShell UI: Talk to a duck!'
date: 2016-01-15T09:00:15+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=404
permalink: /better-know-a-powershell-ui-talk-to-a-duck/
categories:
  - Powershell
  - WPF
tags:
  - Powershell
  - Threads
  - WPF
  - XAML
---
<span style="font-size: small;">If you’ve been following along, you’re in a bit of a pickle. You bought your duck and used the computer to quack at the duck, but now your duck has probably decided the computer is their mom, so it isn’t listening to your voice at all! How do we fix that? Let’s build a UI to make the computer say whatever we want! </span>

<span style="font-size: small;">We are going to start this blog post with an already created UI. If you want to see how I created the UI and generated the PowerShell code, here’s the video of me making the UI:</span>

https://youtu.be/HnpuDw9GuCg

<span style="font-size: small;">Now, here is the code we are starting with:</span>
<pre class="lang:ps decode:true  ">[xml]$xaml = @'
&lt;Window
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Talk_to_a_duck"
        Title="Talk to a duck" WindowStartupLocation="CenterScreen" SizeToContent="WidthAndHeight" &gt;
    &lt;StackPanel Orientation="Horizontal" Margin="5,5,5,5"&gt;
        &lt;Label Content="Enter text to talk to the duck:"/&gt;
        &lt;TextBox Height="23" Text="{Binding Path=TextToTalkToTheDuck}" Width="250" VerticalContentAlignment="Center"/&gt;
        &lt;Button Name="Btn_Speak" Content="Speak!" Width="75" Margin="5,0,0,0"/&gt;
    &lt;/StackPanel&gt;
&lt;/Window&gt;
'@
# Add assemblies
Add-Type -AssemblyName PresentationFramework,PresentationCore,WindowsBase

# Make window
$Window = [Windows.Markup.XamlReader]::Load((New-Object System.Xml.XmlNodeReader $xaml))
$xaml.SelectNodes("//*[@Name]") | Foreach-Object { Set-Variable -Name (("Window" + "_" + $_.Name)) -Value $Window.FindName($_.Name) }
 
$Window_Btn_Speak.Add_Click({
    
})
 
$Null = $Window.ShowDialog()</pre>
If you followed along in the video, or are looking over the XAML code above, you’ll notice in the TextBox I’m using bindings. Bindings are huge in WPF user interfaces, but almost no PowerShell WPF user interfaces use them! Let’s change that!

First off, to access the bindings I need to define the Window DataContext. <a href="https://rachel53461.wordpress.com/2012/07/14/what-is-this-datacontext-you-speak-of/" target="_blank">This article explains</a> what the DataContext is, so go ahead and read that if you want an indepth explanation. Long story short, the DataContext is where all bound variables go when you bind them in XAML. If you don’t define it, they all go to null. If we define it, then we can access anything that is bound! How would we access it? That’s right, custom classes! Man you’re smart!

If you have PowerShell v5, you can create a class easily. Simply use this code:
<pre class="lang:ps decode:true ">Class DuckClass {
    [string]$TextToTalkToTheDuck
}

$DuckDataContext = New-Object -TypeName DuckClass</pre>
The class property needs to be the same name as the binding in the XAML. If you had a UI with 53 different text boxes and 53 different bindings, you could put them all in the same class as long as the properties are all named differently. Now that I have my class, all I have to do is set the $Window.DataContext equal to the DuckDataContext variable. You’ll want to do that sometime between when the window is defined (right under the # Make the window) and when the window is shown. Here’s a snippet where I put it:
<pre class="lang:ps decode:true "># Make window
$Window = [Windows.Markup.XamlReader]::Load((New-Object System.Xml.XmlNodeReader $xaml))
$xaml.SelectNodes("//*[@Name]") | Foreach-Object { Set-Variable -Name (("Window" + "_" + $_.Name)) -Value $Window.FindName($_.Name) }

$Window.DataContext = $DuckDataContext

$Window_Btn_Speak.Add_Click({

})
 
$Null = $Window.ShowDialog()</pre>
I like to put my class definitions at the top, so you’ll see where that is at the end (I don’t want the post to be too long!) Scroll down if you want to see where it is now!

Alright, now that I have the DataContext defined, let’s work with it! In the Add_Click action, let’s output DuckDataContext:
<pre class="lang:ps decode:true ">$Window_Btn_Speak.Add_Click({
    Write-Host $DuckDataContext.TextToTalkToTheDuck
})</pre>
If you run this script, anything you put in the textbox will be sent to the host when you hit the button! If you’re a PS WPF pro, you konw that isn’t very cool. We could just as easily done $TextBox.Text to get the text, so what’s the big deal? I’m getting to that, hold up!

Now, let’s merge the previous posts work and this work. I want to speak to the duck, and I already have this cool code to quack to the duck in another runspace, so let’s merge the two and make this work! Here’s my button code after merging the two:
<pre class="lang:ps decode:true ">$Window.DataContext = $DuckDataContext

Import-Module PoshRSJob
$Window_Btn_Speak.Add_Click({
        $QuackScript = {
        Add-Type -AssemblyName System.speech
        $text = $args[0]
        $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
        $speak.Speak($DuckDataContext.TextToTalkToTheDuck)
    }
    Start-RSJob -Name 'QuackJob' -ScriptBlock $QuackScript
})</pre>
Run the script, type in your text, and you’ll hear….

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-15.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image_thumb-15.png" alt="image" width="540" height="100" border="0" /></a>

Nothing! What happened?

We are trying to access the variable $DuckDataContext in a separate thread! There are a number of different ways to do this, but I usually like to do this with thread safe variables.

Thread safe variables are variables that were designed to move between threads. The nice thing about them is when you update the variable in Thread A, Thread B sees those updates! So we can wrap our class in a thread safe variable by putting it in a synchronized hash table! So, we will want to modify the code a bit to make this happen:

First up, define the DataContext:
<pre class="lang:ps decode:true ">$Windowhash = [HashTable]::Synchronized(@{})
$WindowHash.DuckDataContext = New-Object -TypeName DuckClass</pre>
Now set the Window data context to this new synchrnoized hash table:
<pre class="lang:ps decode:true ">$Window.DataContext = $WindowHash.DuckDataContext</pre>
And lastly, you’ll want to send this threadsafe variable to the new thread with this updated button code:
<pre class="lang:ps decode:true ">Import-Module PoshRSJob
$Window_Btn_Speak.Add_Click({
        $QuackScript = {
        $WindowHash = $args[0]
        Add-Type -AssemblyName System.speech
        $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
        $speak.Speak($WindowHash.DuckDataContext.TextToTalkToTheDuck)
    }
    Start-RSJob -Name 'QuackJob' -ScriptBlock $QuackScript -ArgumentList $WindowHash
})</pre>
Now, what did I change? First off, I need to get the hashtable in the new thread, so I do that with $WindowHash = $args[0]. Then, I need to send the hashtable to the new thread using –ArgumentList in my Start-RSJob. Now, when I run the script, it’s working!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image-16.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image_thumb-16.png" alt="image" width="596" height="127" border="0" /></a>

When you hit the Speak button, it will make your computer speak the text in the textbox! So what am I illustrating? I’m showing that the text being spoken is exactly what is in the text box because of the thread safe variable. We can illustrate this by adding in a sleep:
<pre class="lang:ps decode:true ">$Window_Btn_Speak.Add_Click({
        $QuackScript = {
        $WindowHash = $args[0]
        Start-Sleep 5
        Add-Type -AssemblyName System.speech
        $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
        $speak.Speak($WindowHash.DuckDataContext.TextToTalkToTheDuck)
    }
    Start-RSJob -Name 'QuackJob' -ScriptBlock $QuackScript -ArgumentList $WindowHash
})
</pre>
Here, I am waiting 5 seconds before speaking what is in the text box. Give this a shot, type something into the text box, hit Speak, then quickly change what’s in the text box and hit speak again. Your computer is going to say the same exact thing twice, even though the text changed between when you hit speak the first and second time. Why is it doing this? Because we are using a thread safe variable, so the textbox text will always be accurate no matter what thread you are in!

And that’s it for the duck portion of Better Know a PowerShell UI. Join me next time, when I take this Class idea one step further and add in events! Here's the full finished script:
<pre class="lang:ps decode:true ">Class DuckClass {
    [string]$TextToTalkToTheDuck
}
$Windowhash = [HashTable]::Synchronized(@{})
$WindowHash.DuckDataContext = New-Object -TypeName DuckClass

[xml]$xaml = @'
&lt;Window
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Talk_to_a_duck"
        Title="Talk to a duck" WindowStartupLocation="CenterScreen" SizeToContent="WidthAndHeight" &gt;
    &lt;StackPanel Orientation="Horizontal" Margin="5,5,5,5"&gt;
        &lt;Label Content="Enter text to talk to the duck:"/&gt;
        &lt;TextBox Height="23" Text="{Binding Path=TextToTalkToTheDuck}" Width="250" VerticalContentAlignment="Center"/&gt;
        &lt;Button Name="Btn_Speak" Content="Speak!" Width="75" Margin="5,0,0,0"/&gt;
    &lt;/StackPanel&gt;
&lt;/Window&gt;
'@
# Add assemblies
Add-Type -AssemblyName PresentationFramework,PresentationCore,WindowsBase

# Make window
$Window = [Windows.Markup.XamlReader]::Load((New-Object System.Xml.XmlNodeReader $xaml))
$xaml.SelectNodes("//*[@Name]") | Foreach-Object { Set-Variable -Name (("Window" + "_" + $_.Name)) -Value $Window.FindName($_.Name) }

$Window.DataContext = $WindowHash.DuckDataContext

Import-Module PoshRSJob
$Window_Btn_Speak.Add_Click({
        $QuackScript = {
        $WindowHash = $args[0]
        #Start-Sleep 5
        Add-Type -AssemblyName System.speech
        $speak = New-Object System.Speech.Synthesis.SpeechSynthesizer
        $speak.Speak($WindowHash.DuckDataContext.TextToTalkToTheDuck)
    }
    Start-RSJob -Name 'QuackJob' -ScriptBlock $QuackScript -ArgumentList $WindowHash
})
 
$Null = $Window.ShowDialog()
</pre>