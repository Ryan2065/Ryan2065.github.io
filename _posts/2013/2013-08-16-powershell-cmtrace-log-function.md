---
id: 86
title: Powershell – CMTrace Log Function
date: 2013-08-16T15:10:56+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=86
permalink: /powershell-cmtrace-log-function/
categories:
  - Powershell
tags:
  - cmtrace
  - Log
---
I’ve always been a huge fan of having detailed log files when running scripts at work. I originally wrote a log function to output date &amp; time &amp; message to a file, but it would be hard to read. I decided to write a function in CMTrace format so I could open these files with CMTrace and quickly see what I wanted to see (for instance, Errors show up in red!)

Function:
<pre class="lang:ps decode:true">function Log {
    Param (
		[Parameter(Mandatory=$false)]
		$Message,
 
		[Parameter(Mandatory=$false)]
		$ErrorMessage,
 
		[Parameter(Mandatory=$false)]
		$Component,
 
		[Parameter(Mandatory=$false)]
		[int]$Type,
		
		[Parameter(Mandatory=$true)]
		$LogFile
	)
&lt;#
Type: 1 = Normal, 2 = Warning (yellow), 3 = Error (red)
#&gt;
	$Time = Get-Date -Format "HH:mm:ss.ffffff"
	$Date = Get-Date -Format "MM-dd-yyyy"
 
	if ($ErrorMessage -ne $null) {$Type = 3}
	if ($Component -eq $null) {$Component = " "}
	if ($Type -eq $null) {$Type = 1}
 
	$LogMessage = "&lt;![LOG[$Message $ErrorMessage" + "]LOG]!&gt;&lt;time=`"$Time`" date=`"$Date`" component=`"$Component`" context=`"`" type=`"$Type`" thread=`"`" file=`"`"&gt;"
	$LogMessage | Out-File -Append -Encoding UTF8 -FilePath $LogFile
}</pre>
Parameters are:

Message – Part of the log message
ErrorMessage – Gets put in the log message. If not null, makes the line appear red in CMTrace.
Component – CMTRace has a component column, this will appear in that column
Type – Valid numbers are 1 (Normal), 2 (Warning – Yellow), 3 (Error – Red)
LogFile – The file you want to output to

Usage:
<pre class="lang:ps decode:true">$LogFile = "C:\TestFolder\TestLog.Log"
Log -LogFile $LogFile
Log -Message "This is a normal message" -ErrorMessage $Error -LogFile $LogFile
Log -Message "This is a warning" -Type 2 -Component "Test Component" -LogFile $LogFile
Log -Message "This is an Error!" -Type 3 -Component "Error Component" -LogFile $LogFile</pre>
Output in CMTrace:
<p id="mjhbhPP"><img class="alignnone size-full wp-image-87 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c94ee7e354.png" alt="" /></p>