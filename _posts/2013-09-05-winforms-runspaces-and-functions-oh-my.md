---
id: 317
title: 'WinForms, Runspaces, and Functions &ndash; oh my!'
date: 2013-09-05T19:31:17+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=317
permalink: /winforms-runspaces-and-functions-oh-my/
categories:
  - Powershell
  - WinForms
---
Recently, I came across a <a href="http://learn-powershell.net/2012/10/14/powershell-and-wpf-writing-data-to-a-ui-from-a-different-runspace/">blog post on learn-powershell.net</a> that went over how to write data to a different runspace, and it gave a WPF UI as an example. This gave me an idea, which generally is a dangerous thing. I wondered if I could use this information to create a log function I could place in my user profile to call in scripts when I want nicer logging… I don’t really know how often I’d use something like this, but I had wondered if I could do it, so I had to find out!

The goal of this project was to make two functions. One would pop up a WinForm GUI with a RichTextBox, and the second would push new data to this box so it could function as a log box on the fly. First off, following the article from learn-powershell.net, I created a synchronized hash table and made a new runspace:

 
<pre class="lang:ps decode:true " >Function LogBox {
$Script:SyncHashTable = [Hashtable]::Synchronized(@{})
$RunSpace = [Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace()
$RunSpace.ApartmentState = "STA"
$RunSpace.ThreadOptions = "ReuseThread"
$RunSpace.Open()
$RunSpace.SessionStateProxy.SetVariable("SyncHashTable",$Script:SyncHashTable)
$PowerShellCmd = [Management.Automation.PowerShell]::Create().AddScript({</pre> 


Notice how I used $Script: instead of $?  I did this so the variables would be created on the script level and be accessible by the rest of the script, including other functions!

Now, I’m ready for the WinForm.  I used Primal Forms to create a nice looking form with a text box:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image2-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image2 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image2-1_thumb.png" alt="image2 (1)" width="244" height="207" border="0" /></a>

I then exported the code to my clipboard, and pasted it in a blank script file in PowerGUI. The reason I put it in a blank script file is because I want to replace all of the $ with $Script:SyncHashTable. (the . isn’t a period, it is part of the variable name). After I do this replace (ctrl+F &amp; replace works wonders), and remove the function (Primal Forms automatically wraps all forms in a function, this needs to be removed) I copy the code back into my main script.

 
<pre class="lang:ps decode:true " >[reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
[reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 
$Script:SyncHashTable.form1 = New-Object System.Windows.Forms.Form
$Script:SyncHashTable.TextBox = New-Object System.Windows.Forms.RichTextBox
$Script:SyncHashTable.InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
$Script:SyncHashTable.System_Drawing_Size = New-Object System.Drawing.Size
$Script:SyncHashTable.System_Drawing_Size.Height = 277
$Script:SyncHashTable.System_Drawing_Size.Width = 360
$Script:SyncHashTable.form1.ClientSize = $Script:SyncHashTable.System_Drawing_Size
$Script:SyncHashTable.form1.DataBindings.DefaultDataSourceUpdateMode = 0
$Script:SyncHashTable.form1.Name = "form1"
$Script:SyncHashTable.form1.Text = "Log Box"
 
$Script:SyncHashTable.TextBox.Anchor = 15
$Script:SyncHashTable.TextBox.DataBindings.DefaultDataSourceUpdateMode = 0
$Script:SyncHashTable.System_Drawing_Point = New-Object System.Drawing.Point
$Script:SyncHashTable.System_Drawing_Point.X = 12
$Script:SyncHashTable.System_Drawing_Point.Y = 12
$Script:SyncHashTable.TextBox.Location = $Script:SyncHashTable.System_Drawing_Point
$Script:SyncHashTable.TextBox.Name = "TextBox"
$Script:SyncHashTable.System_Drawing_Size = New-Object System.Drawing.Size
$Script:SyncHashTable.System_Drawing_Size.Height = 253
$Script:SyncHashTable.System_Drawing_Size.Width = 336
$Script:SyncHashTable.TextBox.Size = $Script:SyncHashTable.System_Drawing_Size
$Script:SyncHashTable.TextBox.TabIndex = 0
$Script:SyncHashTable.TextBox.Text = ""
 
$Script:SyncHashTable.form1.Controls.Add($Script:SyncHashTable.TextBox)
 
$Script:SyncHashTable.InitialFormWindowState = $Script:SyncHashTable.form1.WindowState
$Script:SyncHashTable.form1.add_Load($Script:SyncHashTable.OnLoadForm_StateCorrection)
$Script:SyncHashTable.form1.ShowDialog()| Out-Null</pre> 


Now, close off the script and start the new runspace!

 
<pre class="lang:ps decode:true " >})
$PowerShellCmd.Runspace = $RunSpace
$PowerShellCmd.BeginInvoke()
}</pre> 


And that is it for the first function! The log function is pretty easy by comparison:

 
<pre class="lang:ps decode:true " >Function Log ($Message) {
    $Script:SyncHashTable.TextBox.Text = $Script:SyncHashTable.TextBox.Text + "`n" + $Message
}</pre> 


This just adds another line of text to the log box with whatever the message is.  When put together in your profile you’ll get logs in a text box!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image3-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image3 (1)" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image3-1_thumb.png" alt="image3 (1)" width="244" height="140" border="0" /></a>

I’m finding some use for this in my right click tools, but the original idea was to put it in my Powershell profile and have an easy to use WinForm at my fingertips… It’s really not as cool as I thought it’d be.

Here is the full script in case someone wants to see it all put together. If you find a use for it, let me know!

 
<pre class="lang:ps decode:true " >Function LogBox {
$Script:SyncHashTable = [Hashtable]::Synchronized(@{})
$RunSpace = [Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace()
$RunSpace.ApartmentState = "STA"
$RunSpace.ThreadOptions = "ReuseThread"
$RunSpace.Open()
$RunSpace.SessionStateProxy.SetVariable("SyncHashTable",$Script:SyncHashTable)
$PowerShellCmd = [Management.Automation.PowerShell]::Create().AddScript({
 
[reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
[reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 
$Script:SyncHashTable.form1 = New-Object System.Windows.Forms.Form
$Script:SyncHashTable.TextBox = New-Object System.Windows.Forms.RichTextBox
$Script:SyncHashTable.InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
$Script:SyncHashTable.System_Drawing_Size = New-Object System.Drawing.Size
$Script:SyncHashTable.System_Drawing_Size.Height = 277
$Script:SyncHashTable.System_Drawing_Size.Width = 360
$Script:SyncHashTable.form1.ClientSize = $Script:SyncHashTable.System_Drawing_Size
$Script:SyncHashTable.form1.DataBindings.DefaultDataSourceUpdateMode = 0
$Script:SyncHashTable.form1.Name = "form1"
$Script:SyncHashTable.form1.Text = "Log Box"
 
$Script:SyncHashTable.TextBox.Anchor = 15
$Script:SyncHashTable.TextBox.DataBindings.DefaultDataSourceUpdateMode = 0
$Script:SyncHashTable.System_Drawing_Point = New-Object System.Drawing.Point
$Script:SyncHashTable.System_Drawing_Point.X = 12
$Script:SyncHashTable.System_Drawing_Point.Y = 12
$Script:SyncHashTable.TextBox.Location = $Script:SyncHashTable.System_Drawing_Point
$Script:SyncHashTable.TextBox.Name = "TextBox"
$Script:SyncHashTable.System_Drawing_Size = New-Object System.Drawing.Size
$Script:SyncHashTable.System_Drawing_Size.Height = 253
$Script:SyncHashTable.System_Drawing_Size.Width = 336
$Script:SyncHashTable.TextBox.Size = $Script:SyncHashTable.System_Drawing_Size
$Script:SyncHashTable.TextBox.TabIndex = 0
$Script:SyncHashTable.TextBox.Text = ""
 
$Script:SyncHashTable.form1.Controls.Add($Script:SyncHashTable.TextBox)
 
$Script:SyncHashTable.InitialFormWindowState = $Script:SyncHashTable.form1.WindowState
$Script:SyncHashTable.form1.add_Load($Script:SyncHashTable.OnLoadForm_StateCorrection)
$Script:SyncHashTable.form1.ShowDialog()| Out-Null
})
$PowerShellCmd.Runspace = $RunSpace
$PowerShellCmd.BeginInvoke()
}
 
Function Log ($Message) {
    $Script:SyncHashTable.TextBox.Text = $Script:SyncHashTable.TextBox.Text + "`n" + $Message
}</pre> 
