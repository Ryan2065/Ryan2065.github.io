---
id: 414
title: 'Better Know a PowerShell UI: 50 Shades of Bindings (Part 1)'
date: 2016-01-18T21:30:35+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=414
permalink: /better-know-a-powershell-ui-50-shades-of-bindings-part-1/
categories:
  - Powershell
  - WPF
tags:
  - Classes
---
I wanted to take some time to go over bindings in WPF and how they can be used with classes to simplify and optimize your code. At this moment, events do not work in PowerShell classes, so in order to do what I’m about to do, we need to create classes in C# and bring them over to PowerShell. At the end of this post, I’ll show code I use to automatically do this, so don’t worry about having to code in C#! You’ll be able to just throw code at a function and BAM – class!

Bindings in WPF only really save you a lot of time when you are making a threaded user interface. You cannot access WPF controls from another thread, so you have to pass data from your working thread to your UI thread, and then have something in the UI thread trigger the update in order to change data in the UI. In order to get around this, we can create a custom class, create a synchrnoized hashtable, and then put the class in the synchrnoized hash table. With this combination, we can now modify bound properties of the WPF UI in a separate thread without having to do magic in the UI thread!

So, can we use the new classes in PowerShell 5? Yes and no… Custom PowerShell classes work the same as .Net classes so you can use them, but eventing doesn’t work in PowerShell classes, so you can’t use all the cool features of binding. In plain English, you can really only use PowerShell classes to get information from the user, not display information to them after doing some work. Because of this limitation, I only use C# classes, and I’m going to show you how to easily create a C# class!

First off, let’s add in the PowerShell code to create the class:

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'

'@</pre> 


Now, we are ready to code in C#! Add in the below using line. We want to create a class that has a specific event in it, so we need to use <a href="https://msdn.microsoft.com/en-us/library/system.componentmodel(v=vs.110).aspx" target="_blank">System.ComponentModel</a>.

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'
using System.ComponentModel;

'@</pre> 


Now we need to create a class that can be used by PowerShell. So, we create a <strong>public class</strong>! Name the class whatever you want, in the example I named it WindowClass. The last part of the creation of the class is INotifyPropertyChanged. This means the class inherits from INotifyPropertyChanged which will allow us to create events to update the WPF UI:

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'
using System.ComponentModel;

public class WindowClass : INotifyPropertyChanged
{

}
'@</pre> 


Now, let’s create a property of the class. The class properties are whatever you decide to bind to in WPF. For this example, I’m going to make the window title bound to the property WindowTitle by setting Title=”{Binding Path=WindowTitle}”. There are two parts to each property, the private variable and public variable. The private variable is something internal to the class that only the class uses. The public variable is the actual name of the variable. These cannot be equal, but since case matters in C#, you can make the private variable be windowtitle, and the public variable be WindowTitle. WindowTitle is also going to be a string, so I want to make it a string. If I didn’t know the type, I could simply say object instead of string:

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'
using System.ComponentModel;

public class WindowClass : INotifyPropertyChanged
{
    private string windowtitle;
    public string WindowTitle
    {

    }
}
'@
</pre> 

Now, I need to tell .Net what to do when I set WindowTitle. To do this, I need to call get, and then set. This gets the value I’m sending it, and sets the property to the value. Now, here is where we add the event. If I simply put in get and set, I’m going to update the variable, but will not tell the UI it was updated, so the UI won’t update! In order to add in the event, I need to call the method NotifyPropertyChanged whenever I set the variable:

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'
using System.ComponentModel;

public class WindowClass : INotifyPropertyChanged
{
    private string windowtitle;
    public string WindowTitle
    {
        get { return windowtitle; }
        set
        {
            windowtitle = value;
            NotifyPropertyChanged("WindowTitle");
        }
    }
}
'@</pre> 


Alright, ALMOST done. We now simply need to define NotifyPropertyChanged, and we will be ready to go! In this code, I’ve added the NotifyPropertyChanged method. We are creating a public event (if it wasn’t public, the UI wouldn’t know about it) and using a private method to trigger it (because nothing outside of the class needs to access the method). When we called NotifyPropertyChanged, we passed the name of the property changing (in this case, WindowTitle). Looking at the code, we are saying if the parameter property is not null, it is going to raise the event PropertyChanged:

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'
using System.ComponentModel;

public class WindowClass : INotifyPropertyChanged
{
    private string windowtitle;
    public string WindowTitle
    {
        get { return windowtitle; }
        set
        {
            windowtitle = value;
            NotifyPropertyChanged("WindowTitle");
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;
    private void NotifyPropertyChanged(string property)
    {
        if(PropertyChanged != null)
        {
            PropertyChanged(this, new PropertyChangedEventArgs(property));
        }
    }
}
'@</pre> 


And that’s it! You now have a class that uses an event! If I want to add a second property of an unknown type (so it can be anything), it’s pretty easy. Simply use the WindowTitle as the template and add a new one. In this example, I added the new property Property1:

 
<pre class="lang:ps decode:true " >Add-Type -Language CSharp @'
using System.ComponentModel;

public class WindowClass : INotifyPropertyChanged
{
    private string windowtitle;
    public string WindowTitle
    {
        get { return windowtitle; }
        set
        {
            windowtitle = value;
            NotifyPropertyChanged("WindowTitle");
        }
    }

    private object property1;
    public object Property1
    {
        get { return property1; }
        set 
        {
            property1 = value;
            NotifyPropertyChanged("Property1");
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;
    private void NotifyPropertyChanged(string property)
    {
        if(PropertyChanged != null)
        {
            PropertyChanged(this, new PropertyChangedEventArgs(property));
        }
    }
}
'@</pre> 


To actually use this new class, I simply use <a title="Read online help for this command" href="http://go.microsoft.com/fwlink/?LinkID=113355" target="_blank">New-Object</a>:

 
<pre class="lang:ps decode:true " >$Test = New-Object -TypeName WindowClass</pre> 


How do I create classes? I use my function <a title="WPF Coder!" href="http://www.ephingadmin.com/wpf-the-superduper-easy-way/" target="_blank">New-EphingWPFCode</a> and use the –CreateClass parameter when sending in XAML. This will parse the XAML and look for bindings, then create your class based on the bindings it finds!

That's it for this time! Join me next time when I'll show you how to use this custom class in a thread safe variable!