---
id: 714
title: 'Collectionless Deployments'
date: 2019-02-17
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=714
permalink: /CollectionlessDeployments/
categories:
  - SCCM
hidden: true
---

# SCCM Collectionless* Deployments

Collectionless deployments are finally here! Well, kinda. They still involve collections and as of 1810, they aren't really ready for production. Let's dig into them!

First off, what do I mean when I say collectionless deployments? The goal of collectionless deployments is to get rid of the need for Application install collections. Are you or is someone you know guilty of the following collection structure?

```text
-Root
  -Installs
    -Java v6 - Install
    -Adobe Shockwave - Install
    -Silverlight - Install
  -Uninstalls
    -Java v6 - Uninstall
    -Adobe Shockwave - Uninstall
    -Silverlight - Uninstall
```

Microsoft wants you to stop the madness and get rid of these types of collections! In order to help facilitate this, they decided to bring user approvals to devices!

![Confused](https://media.giphy.com/media/h4Z6RfuQycdiM/giphy.gif)

Ok ok, it's not as weird as it sounds and it's actually a really good way to handle this!

So here's the idea, deploy the application as "Requires Approval" to a collection of devices. Then, the site server holds that deployment and doesn't tell the devices about it until someone goes in and approves it! 

Why would we want to do something like that? Instead of the collection structure above, Microsoft would prefer we have collection structures like this:

```text
-Root
  -Departments
    -HR
    -Finance
  -Models
    -Dell
    -HP
    -Lenovo
```

We can now get away with making logical grouping of computers that isn't centered around software. You still *can* if you want, but you no longer *have* to.

Do all my HR computers need Candy Crush? Cool, deploy it to the HR collection now and save yourself the hassle of creating and maintain a Candy Crush collection! Hey, this one guy in Finance wants Candy Crush but isn't in the HR collection? Not a problem, create an approval for his computer and it'll install without any collection membership changes!

See, the goal of this isn't to become the new way to deploy in most cases, but instead kill off those "one-offs" that were forcing us to have install collections for everything.

Alright, now that we know why it exists and what it can do for us, how could I use it?

First off, Microsoft says the feature is SDK only right now which is a nice way of saying they haven't had time to put it in the UI yet! So, let's look at the PowerShell you'd need to make one of these deployments:

```PowerShell
    $Deployment = New-CMApplicationDeployment `
                    -CollectionName 'All Systems' `
                    -Name $Application.LocalizedDisplayName `
                    -DeployAction Install `
                    -DeployPurpose Available `
                    -ApprovalRequired $true
```

Alright, first 