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

Collectionless deployments are finally here! Well, kinda. They still involve collections and as of 1810, they aren't really ready for production. First I'll talk about what they are, show you how to use them, and then talk about some shortcomings that make me not want to use them in production yet.

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

So here's the idea, deploy the application as "Requires Approval" to a collection of devices. Then, the site server holds that deployment and doesn't tell the devices about it until someone (say, a field tech) goes in and approves it! 

Why would we want to do something like that? Instead of the collection structure above, we could instead have this collection structure:

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

Now that we know why it exists and what it can do for us, how could I use it?

First off, Microsoft says the feature is SDK only right now which is a nice way of saying they haven't had time to put it in the UI yet! So, let's look at the PowerShell you'd need to make one of these deployments:

```PowerShell
    $Deployment = New-CMApplicationDeployment `
                    -CollectionName 'All Systems' `
                    -Name $Application.LocalizedDisplayName `
                    -DeployAction Install `
                    -DeployPurpose Available `
                    -ApprovalRequired $true
```

This is a generic PowerShell CM Application Deployment command, but the big difference is "ApprovalRequired" is set to true on a Device deployment. After creating this deployment, what happens on the computers in that small collection of "All Systems"? *Nothing*! Devices won't receive policy until the approval itself happens, so no installs, no detection methods running, and no notifications to your end users that new software is in Software Center.

How do you approve? Here you go:

```PowerShell
$machine = Get-WmiObject `
    -Namespace 'root\sms\site_CHQ' `
    -Query "SELECT * FROM SMS_R_SYSTEM WHERE Name = '$($env:COMPUTERNAME)'"

$clientGuid = $machine.SMSUniqueIdentifier
$appid = $Application.ModelName
$autoInstall = "true"
$comments = "Approved"

Invoke-WmiMethod `
          -Path "SMS_ApplicationRequest" `
          -Namespace 'root\sms\site_CHQ' `
          -Name CreateApprovedRequest `
          -ArgumentList @($appid, $autoInstall, $clientGuid, $comments)
```

Now that an approval happens, CM uses the fast channel to send the policy to the computer and start the install! If the computer is not on, it will get the policy when it next turns on through normal means.

What else can you do with this? If you deny the approval (available in the console or PowerShell), then the application will uninstall on the device. You can also re-approve the approval to reinstall with this command:

```PowerShell
$reqObj = Get-WmiObject -Namespace 'root\sms\site_CHQ' -Class SMS_UserApplicationRequest | `
    Where {$_.ModelName -eq $appid -and $_.RequestedMachine -eq $machinename }
$reqObj.Approve('Approved', 1)
```

You might be thinking to yourself, cool, let's start using these in production. Hold your horses! There are a number of bugs and/or design issues right now that will probably keep you from using this outside of a POC:

1) The WMI method to approve sometimes throws errors when it's successful
2) If you edit the device "Requires Approval" deployment in the UI, it will remove the "Requires Approval" flag and the deployment will go out to your environment as Available
3) If you run the New-CMApplicationDeployment command with DeployPurpose=Required and requires approval, it'll create a required deployment without the requires approval flag
4) If the application fails to install the first time (no content on DP, you're testing installs with this, etc), it will never try to re-install. You have to deny the approval and run the re-approve command to reinstall.

I think this is a step in the right direction, and this feature has the potential to be the coolest thing to happen to collections since Include/Exclude rules. I'll be watching the progress of this feature closely, but won't start using it in production just yet.

