---
id: 212
title: 'Better Know a PowerShell UI: Let&#8217;s go buy a duck! (Part 1)'
date: 2016-01-12T09:00:51+00:00
author: Ryan Ephgrave
layout: post
guid: https://www.ephingadmin.com/?p=212
permalink: /better-know-a-powershell-ui-lets-go-buy-a-duck/
categories:
  - WPF
tags:
  - XAML
---
Welcome to the third installment of Better Know a PowerShell UI! In today’s post we will build our first UI to assist us in buying a duck. The tools I will be using are the PowerShell ISE, Visual Studio, and the ISE addon run in a new session. I will not use my function that automatically generates the WPF code for you because then how will you learn?

First up, open Visual Studio and select New Project. You’ll want to select Visual C# and then WPF Application. Name the project, make sure the project is being stored in a good location for you, and then click Ok!

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb.png" alt="image" width="1028" height="487" border="0" /></a>

Let’s go over what you should be seeing. On the left side, you should see the toolbox tab:

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-1.png" alt="image" width="281" height="484" border="0" /></a>

If the toolbox is expanded, you’ll see this:

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-2.png" alt="image" width="564" height="484" border="0" /></a>

The toolbox houses all the built in WPF controls (text box, labels, spreadsheet, etc). While you are still learing, you’ll probably want to use this to select the controls you want to add.

If you don’t see the toolbox, make sure it is checked. Go to View and scroll halfway down. You should see Toolbox. Select that. If something suddenly went away, that was the toolbox! If something appeared, this is the toolbox!

On the right side, you should see the Solution Explorer.

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-3.png" alt="image" width="410" height="374" border="0" /></a>

Because you are technically making a C# program right now, there are other parts to it than the xaml. If a cat runs across your keyboard and closes your designer (or it closes for any other number of plausable reasons) you can re-open it by double clicking on MainWindow.xaml.

Also on the right side, you should see the Properties tab (in the picture above, it’s the lonely tab in the upper right). You’ll come to love this tab, as it lists all the available properties and events available for each control you place on the window.

In the center of the screen you should see the designer:

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-4.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-4.png" alt="image" width="644" height="427" border="0" /></a>

Here you’ll be designing the window! It’s pretty self explanatory…

Lastly, you should also see the design window. This window has all the code of the WPF window in it. As you design your window, the code will update to reflect what are you designing above. As you get better at this, you’ll be able to design completely through the code and only use the above window to see what your changes look like. Baby steps… For now we’ll ignore the code until the end!

So, what did I say we were going to do today? Oh, right, a duck dash button! If you don’t know about the Amazon Dash button, I like to think of it as an emergency button for common household products. Out of laundry detergent? Hit the dash button! Out of toilet paper? Dash! Amazon has yet to make a dash button for ducks, so we’ll try our best to fix that.

Now, when we design our duck dash button, we will obviously need a button. So, go to your tool box, expand “Common WPF Controls”, and double click on button.

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-5.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-5.png" alt="image" width="644" height="176" border="0" /></a>

You should now have a button on your WPF window and the code for the button will magically appear on the bottom. Lets move this button to the center of the screen. Click and drag!

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-6.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-6.png" alt="image" width="644" height="423" border="0" /></a>

Now, on the right side open up the properties tab and look at what we have. If we change the name of the button to “Duck Dash!” you’ll see it’s auto corrected to Duck_Dash_. Think of name as variable name. The name is how you will programmatically interact with the object and is only used on the backend. As a general rule, I only name things I plan on interacting with. Because I want something to happen when I click on the button, I want to name this! I’ll name it DuckButton.

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-7.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-7.png" alt="image" width="426" height="126" border="0" /></a>

So, how do we change the text in the center of the button? Well this is what Content is for. A button’s content is the text inside the button. Let’s change the text to “Dash!”

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-8.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-8.png" alt="image" width="244" height="145" border="0" /></a>

And see our button update to the left:

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-9.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-9.png" alt="image" width="244" height="112" border="0" /></a>

Since my button simply says dash, I want to add something in to tell me what I’m dashing. On the left side in the toolbox, double click on Label. Now, drag the label to somewhere appropriate, and change the content to something descriptive with the Property tab on the right side.

<a href="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image-10.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://secure.bluehost.com/~ephingad/wp-content/uploads/2016/01/image_thumb-10.png" alt="image" width="244" height="133" border="0" /></a>

I almost never name labels because I don’t usually interact with them. It doesn’t hurt anything to name them, so we can just leave it named Label.

And now you’ve designed your Duck Dash user interface! Join me next time when we take this code and make it do something in PowerShell!