---
id: 660
title: Pester Testing .Net with PowerShell Classes
date: 2016-09-28T22:07:00+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=660
permalink: /pester-testing-net-powershell-classes/
categories:
  - Powershell
tags:
  - .Net
  - Pester
  - Powershell
---
<span style="font-size: small;">One problem I’ve come across with Pester is it has no good way to Mock .Net objects. This hasn’t caused me too much trouble as I can just wrap .Net methods in functions and mock the function, but I had a case come up where I needed to specify the .Net type in my function parameter, so I couldn’t just wrap this in a function:</span>
<pre class="lang:ps decode:true">Function New-BadNetFunction {
    Param (
        [Parameter(Mandatory=$true)]
        [Breaks.Pester.MyType]$MyType
    )
    returns $Bad
}</pre>
<span style="font-size: small;">As you can see, I’m specifying the type and it is a required parameter, so how do I get around this? I could just use [Object] as the type, but that seems wrong. I know classes now exist in PowerShell 5, so I looked into creating a Breaks.Pester.MyType class, but that isn’t possible with PowerShell 5 classes. I could use PowerShell 5 classes to create MyType, so is there a way to get rid of Breaks.Pester?</span>

<span style="font-size: small;">It’d be a short blog post if there wasn’t! PowerShell 5 brought with it the idea of namespaces. You can stick these at the beginning of your code to simplify .net names. So if I just stick this at the start of my script (literally has to be line 1 and 2):</span>
<pre class="lang:ps decode:true ">#requires -Version 5.0
using namespace Breaks.Pester</pre>
<span style="font-size: small;">I can now simplify my function to:</span>
<pre class="lang:ps decode:true ">Function New-GoodNetFunction {
    Param (
        [Parameter(Mandatory=$true)]
        [MyType]$MyType
    )
    returns $Good
}</pre>
<span style="font-size: small;">And now I can simply create a class in Pester and test this function with this test:</span>
<pre class="lang:ps decode:true crayon-selected">Class MyType {
    $Name
}

Describe 'CanTest' {
    It 'ActuallyTests!' {
        $Test = New-Object MyType
        New-GoodNetFunction -MyType $Test
    }
}</pre>