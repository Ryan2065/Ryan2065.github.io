---
id: 711
title: 'PowerShell Script Analyzer Community Rules in VS Code'
date: 2018-02-19T01:10:39+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=711
permalink: /community-script-analyzer-rules-in-code/
categories:
  - PowerShell
  - VSCode
---

# PowerShell Script Analyzer Community Rules in VS Code

I have been using [VS Code](https://code.visualstudio.com/) for a little over 
a year now and just found a cool new-ish feature! 

If you've ever been coding a PowerShell script in Code and have noticed the 
squigly lines under a variable, you've seen the module PSScriptAnalyzer working:

![PSScriptAnalyzer Working](..\images\2018-02-19\EphingAdmin-0000.jpg)

I was interested in adding some custom rules, so I wanted to see if I could add
the community PSScriptAnalyzer rules to the mix. 

First, I went and got the community script analyzer rules. These are maintained
by Microsoft in the [PSScriptAnalyzer repo](https://github.com/PowerShell/PSScriptAnalyzer) 
under the obvious path of PSScriptAnalyzer/Tests/Engine/CommunityAnalyzerRules:

![Community Rules Path](..\images\2018-02-19\EphingAdmin-0001.jpg)

I just downloaded the repository and then copied that folder to my scratch work folder.

You will now need to create a file to store the PSScriptAnalyzer settings. 
VSCode uses settings files to control the different settings of this module
and we can provide an override. Let's keep it simple and use this file:

```
@{
    CustomRulePath = 'C:\Users\Ryan\source\ScratchWork\CommunityAnalyzerRules'
    IncludeDefaultRules = $true
}
```

Save this as CustomRules.psd1 somewhere on your computer. Make sure to change 
the path specified to the location where you downloaded CommunityAnalyzerRules.

Now, in VSCode click File -> Preferences -> Settings

![Code Settings](..\images\2018-02-19\EphingAdmin-0002.jpg)

Search for PowerShell.ScriptAnalysis.settingsPath and click the edit button:

![Code Settings](..\images\2018-02-19\EphingAdmin-0003.jpg)

Add in the full path to the .psd1 file you just created. Here is my setting:

```
"powershell.scriptAnalysis.settingsPath": "C:\\Users\\Ryan\\source\\ScratchWork\\CustomRules.psd1"
```

Notice this is JSON so we have to escape the \\ mark with two of them.

Save your settings, then either close VS Code and re-open it, or just nuke the
PowerShell Integrated Console terminal window and re-open it. You'll now have 
access to the PowerShell community script analyzer rules!

![Code Settings](..\images\2018-02-19\EphingAdmin-0004.jpg)

If you want to have a look at other options in the settings files, you can
see some examples in the PowerShell extension folder on your computer! Open
the path:

```
C:\Users\Ryan\.vscode\extensions\ms-vscode.powershell-1.5.1\modules\PSScriptAnalyzer\Settings
```

Changing Users\Ryan to your user profile. This is where VSCode stores the default
rule sets. Feel free to play around with the files to get them to include only
the rules you care about, and start creating and using your own rules!

**As of Feburary 2018, there seems to be a bug in the PSScriptAnalyzer settings
file. If you specify the CustomRulePath like we do, you can no longer pick
and choose your enabled rules. It's all or nothing. This may or may not be your
desired outcome. Just know that we cannot specify the option IncludeRules.
Hopefully this will be fixed soon!
