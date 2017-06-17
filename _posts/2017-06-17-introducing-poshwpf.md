---
id: 709
title: 'Introducing PoshWPF!'
date: 2017-6-17T01:10:39+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=709
permalink: /introducing-poshwpf/
categories:
  - PowerShell
  - WPF
---

# Introducing PoshWPF!

I've been working on PowerShell UIs for awhile now and have always copied and pasted 
code to show the UI and thread it. I decided to take my knowledge and publish a PowerShell 
module to make this process easier!

To install this module, simply run the code:

```PowerShell
Install-Module PoshWPF
```

If you want to display a WPF window, design it in Visual Studio (use Community 
Edition if you don't want to pay) and run the command:

```PowerShell
New-WPFWindow -xaml $xaml
```

This will open the UI in a separate thread which will allow you to interact with it 
at the PowerShell prompt.  Here's a complete working example:

```PowerShell
Import-Module PoshWPF
$xaml = @'
<Window x:Class="WpfApp1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfApp1"
        mc:Ignorable="d"
        Title="My Window Title" Height="350" Width="525">
    <StackPanel>
        <Button Name="Button" Width="75" Height="30" Content="Button"/>
        <TextBox Name="TextBox"/>
    </StackPanel>
</Window>

'@

New-WPFWindow -xaml $xaml

```

![New-WPFWindow](\images\2017-6-17\New-WPFWindow.jpg)

If you notice, I didn't even modify the Visual Studio XAML code to remove x:Class. 
The module will do that for you!

You'll be able to interact with any named objects through the PowerShell prompt. 
If you want to view the properties of an object, simply run the command:

```PowerShell
Get-WPFControl -ControlName 'Button'
```

And you'll be given a hash of the properties. You are not returned the object, 
because if you try to edit it from another thread everything will crash. 

If you would like to edit one of the controls, simply run the command:

```PowerShell
Set-WPFControl -ControlName 'Button' -PropertyName 'Content' -Value 'NewValue'
```

![ChangeButtonValue](\images\2017-6-17\ChangeButtonValue.jpg)

If you want to subscribe to an event in the window, use the command:

```PowerShell
New-WPFEvent -ControlName 'Button' -EventName 'Click' -Action { Write-Host 'Clicked!' }
```

When you click the button you'll now see 'Clicked' in your host!

![Clicked](\images\2017-6-17\Clicked.jpg)

Lastly, if you'd like to do something more advanced in your UI window, you can 
use the command:

```PowerShell
Invoke-WPFAction -Action { $Global:PoshWPFHashTable.WindowControls.Window_TextBox.Text = 'My New Text!!!' }
```

This command will add new text to the textbox! I could have done that with 
Set-WPFControl, but I wanted to show how you could interact with the UI in the 
other thread:

![ChangeText](\images\2017-6-17\ChangeText.jpg)

Please play around with the module and let me know what you think!