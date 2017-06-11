---
id: 309
title: 'Powershell Function: Merge Log Files'
date: 2013-10-07T19:25:54+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=309
permalink: /powershell-function-merge-log-files/
categories:
  - ConfigMgr
  - Powershell
---
I’ve been working on a script to try and trace an application install through the log files of a client. So far, I’ve been able to write a function that will merge all the log files together into an array list, and then sort them based on date. I thought others might like to use it, so I figured I’d post the function and give an example of how to search through it.  To run through an entire CM2012 client log directory, merging every log file and sorting by date, it takes about 2 minutes… Not that bad considering there are 5 smsts logs in the directory I scanned!

The usage is, create an array of log files (full path) and then pass it to the function. It will create an array list called LogArray that can be accessed throughout the rest of your script.
<pre class="lang:ps decode:true ">Function MergeLogFile {
    Param (
		[Parameter(Mandatory=$true)]
		$LogFiles
	)
	$Script:LogArray = New-Object System.Collections.ArrayList
	Foreach ($LogFile in $LogFiles) {
		$count = 0
		if (Test-Path $LogFile) {
			$LogName = $LogFile.Split("\")
			$LogName = $LogName[$LogName.length - 1]
			$LogName = $LogName -replace ".log",""
			$LogName = $LogName.split("-")
			$LogName = $LogName[0]
			$LogContent = Get-Content $LogFile
			foreach ($Line in $LogContent) {
				$LogMessage = $null
				if ($Line.ToLower().contains("&lt;![log[")) {
					$LogMessage = $Line
					$EndLoop = $false
					$LoopCount = 0
					Do {
						$LoopCount++
						if (!($LogMessage.ToLower().contains("]log]!&gt;"))) {$LogMessage = $LogMessage + "`n" + $LogContent[$count + $LoopCount]}
						else {$EndLoop = $true}
						if ($LoopCount -gt 5000) {$EndLoop = $true}
					} while ($EndLoop -ne $true)
				}
				if ($LogMessage -ne $null) {
					$MessageCount = 0
					$PropStart = 0
					$LogComponent = $null
					$LogType = $null
					$Message = $null
					$PropStart = $LogMessage.IndexOf("&lt;time=")
					$PropEnd = $LogMessage.Length
					$PropText = $LogMessage.Substring($PropStart)
					$PropSplit = $PropText.Split("`"")
					foreach ($instance in $PropSplit) {
						$MessageCount++
						if ($instance.contains("&lt;time=")) {
							$LogTime = $PropSplit[$MessageCount]
							$LogDate = $PropSplit[$MessageCount+2]
						}
						elseif ($instance.contains("component=")) {$LogComponent = $PropSplit[$MessageCount]}
						elseif ($instance.contains("type=")) {$LogType = $PropSplit[$MessageCount]}
					}
					$DateTime = $null
					$LogTime = $LogTime.split("+")
					$LogTime = $LogTime[0]
					$DateTime = [DateTime]"$LogDate $LogTime"
					$MessageStart = $LogMessage.IndexOf("&lt;![LOG[") + 7
					$MessageEnd = $LogMessage.IndexOf("]LOG]!&gt;") - 7
					$Message = $LogMessage.Substring($MessageStart,$MessageEnd)
					$PSObject = New-Object PSObject @{
					Message = $Message
					DateTime = $DateTime
					OriginalMessage = $LogMessage
					}
					$Script:LogArray.Add($PSObject) | Out-Null
				}
				$count++
			}
		}
	}
	$Script:LogArray = $Script:LogArray | Sort-Object {$_.DateTime}
}</pre>
If you want to search through this log array, you could use this loop to search:
<pre class="lang:ps decode:true ">$Search = "Search String"
foreach ($instance in $Script:LogArray) {
    $Message = $instance.Get_Item("Message")
	if ($Message.contains("$search")) {
		$LogFile = $instance.Get_Item("LogName")
		$LogMessage = $instance.Get_Item("OriginalMessage")
		$LogMessage | Out-File -Append -Encoding UTF8 -FilePath $SaveFile
	}
}</pre>
This will find all messages that include the search string $Search and will save the lines into the file $SaveFile. This way you can just open the saved file with CMTrace. This is why it exports the file in UTF8. CMTrace can’t handle ANSI.