---
id: 725
title: 'Passwordless PowerShell'
date: 2021-02-06
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=725
permalink: /PasswordlessPowerShell/
categories:
    - PowerShell
---

# Passwordless PowerShell - How to use gMSAs In Your Scripts

One thing I've always hated doing with PowerShell is storing/retrieving passwords, because that always feels like a weak link in the security chain. I love running code as multiple accounts (my team probably has 75+ AD accounts we own), but hate everything else we have to do with those accounts.

A few years ago we heard about these things called gMSAs. They are accounts, managed by Active Directory, and are passwordless (not really, but you don't have to care about the password)! Instead of getting a traditional password, you tell AD who is allowed to use that password, and then they can use the credential whenever they want! Sounds awesome, right?

![Awesome](https://media.giphy.com/media/VGthqYKqyKhipYxK2s/giphy.gif)

There's a catch. They were made to be used by Windows and don't have great support outside of that use case. I'm not Windows.

![Quit playing games](https://media.giphy.com/media/qcoocjBhD5Zle/giphy.gif)

So, years ago a team member (Jeff Scripter) wrote a function to get a PSCredential object from a gMSA. This work was based on the open source tool [DSInternals](https://github.com/MichaelGrafnetter/DSInternals) which has functionality built in to get the password.

After seeing some talk on Twitter about gMSAs, I decided to write a module so everyone can use them like our team does!

![Alright](https://media.giphy.com/media/t75AqiyT97TmIZZf3V/giphy.gif)

I'm not going to go over setting up gMSAs, mostly because I already did that in the [documentation](https://github.com/Ryan2065/gMSACredentialModule) for the module (at the bottom). I'm also not going to go super in-depth into the module, because it's pretty small. Instead, I'm going to talk about how to set up your environment so you can achieve passwordless PowerShell!

First off, you'll want to decide how you'll get access to your script passwords. Your choices are realistically "AD User" or "AD Group". We've set things up with an AD group and only one user in it, but do whatever makes you happy.

Now that you have your password retriever, make your gMSAs!

``` PowerShell
$ADGroupName = 'Not Password Retrievers' # to fool the hackers
$GMSAName = 'gMSASQLRead'
$DomainFqdn = 'home.lab'
$ServiceAccount = New-ADServiceAccount -Name $GMSAName -DNSHostName "$GMSAName.$($DomainFqdn)" -PrincipalsAllowedToRetrieveManagedPassword $ADGroupName -Enabled $true
```

With the above code, any AD object (computer or user) in the group "Not Password Retrievers" will be able to get the gMSA password. So just drop your PowerShell service account in that group.

Next, use it!

``` PowerShell
Install-Module GMSACredential
$Cred = Get-GMSACredential -GMSAName 'gMSASQLRead' -Domain 'Home.Lab'
$Results = Invoke-GMSACommand -Credential $Cred -ScriptBlock {
    # Code to query remote SQL server
}
```

So there's a little to unpack in the above code. First, line 1 installs the module. Ok, we're starting out easy.

Next, we get the gMSA credential from AD. This line will only work if you are running as the user who can get the password. 

Lastly, I'm running Invoke-GMSACommand (which is based on Invoke-Command). Why am I not just running Invoke-Command? Invoke-GMSACommand creates a token for the gMSA account and then executes Invoke-Command as that token. This works *extremely* well when you are accessing resources off-box like a remote SQL server and you can't run a remote PowerShell command against that server. Almost all my team's scripting is done against remote servers so this is what we always use. 

*Note* if you only want to do something locally (ie, I'm on Computer1 and I want to access resources on Computer1) - feel free to just use Invoke-Command -ComputerName localhost -Credential $Cred.

Aaaand that's all there is to it! There's not a lot to this module, just one command to get a credential and another to use it. 

Please let me know if this does or does not work in your environment (works in my lab) or if you have any suggestions for making it better, comment or leave a note in [GitHub](https://github.com/Ryan2065/gMSACredentialModule/issues), or find me on [Twitter](https://twitter.com/ephingposh?lang=en)

Thanks for reading, and remember

![BackstreetsBack](https://media.giphy.com/media/65GiuFnyEuMjm/giphy.gif)
