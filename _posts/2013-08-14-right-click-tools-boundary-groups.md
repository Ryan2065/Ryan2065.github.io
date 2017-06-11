---
id: 81
title: Right Click Tool Preview – Boundary Groups
date: 2013-08-14T14:55:30+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=81
permalink: /right-click-tools-boundary-groups/
categories:
  - Powershell
  - Right Click Tools
tags:
  - Boundary
  - Boundary Group
  - Console Extension
  - Subnet
---
It has been awhile since I’ve posted any updates on the Powershell Right Click Tools, so I thought I’d go through the new tool to find boundary groups. Here you’ll find the powershell code I use to figure out what boundary groups a computer is in, and if there is a overlap. I’m still a few weeks out from releasing the newest version, so this is just a quick preview of what I’m working on. Over the next few weeks, I’ll post more previews to showcase some of the new tools I’ve been dreaming up!

I’ve seen a number of posts over the past few months where people spend hours trying to figure out a problem, and it winds up being due to overlapping boundary groups. I decided I could write a tool and put it in the upcoming right click tools to help identify overlapping boundaries for a specific computer.

There will be a new device tool called “Client Information”
<p id="kcmLiVl"><img class="alignnone size-full wp-image-83 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c91f1cbf21.png" alt="" /></p>
Here you’ll be able to find general information about the client installed on the computer (Version, site code, components, cache, applications, and execution history) and I just decided to add a tab for boundaries!

&nbsp;
<p id="zCHwbTS"><img class="alignnone size-full wp-image-84 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c920a0227a.png" alt="" /></p>
Alright, how do I do this?  First off, I don’t do anything with IPV6. I’m not even going to pretend I know how to do stuff with IPV6 in Powershell. That will have to wait for another release.

I created two functions to help me with this process. The first is called GetSubnet. You give it an IP Address and a Subnet Mask, and it returns the subnet:
<pre class="toolbar:1 lang:ps decode:true">Function GetSubnet ($IPAddress, $SubnetMask) {
	$IPAddress.split(".") | %{$IPAddressBinary = $IPAddressBinary + $([Convert]::ToString($_,2).PadLeft(8,"0"))}
	$SubnetMask.split(".") | %{$SubnetMaskBinary = $SubnetMaskBinary + $([Convert]::ToString($_,2).PadLeft(8,"0"))}
	$IPBits = $SubnetMaskBinary.indexOf("0")
	$ipBinary = $($IPAddressBinary.substring(0,$IPBits).padright(32,"0"))
	do {$IPDottedDec += "." + [String]$([Convert]::ToInt32($ipBinary.substring($i,8),2)); $i+=8} while ($i -le 24)
	$Subnet = $IPDottedDec.substring(1)
	return $Subnet
}</pre>
The second function is called BoundaryGroup. You give it the ID of a boundary you know is in a boundary group, and it adds the boundary group information into an array:
<pre class="toolbar:1 lang:ps decode:true">Function BoundaryGroup ($BoundaryID) {
	$strQuery = "Select * from SMS_BoundaryGroupMembers join SMS_BoundaryGroup on SMS_BoundaryGroup.GroupID = SMS_BoundaryGroupMembers.GroupID where SMS_BoundaryGroupMembers.BoundaryID like '$BoundaryID'"
	Get-WmiObject -Query $strQuery -Namespace $Namespace -ComputerName $Server | ForEach-Object {
		$GroupName = $_.SMS_BoundaryGroup.Name
		$GroupCode = $_.SMS_BoundaryGroup.DefaultSiteCode
		$GroupDescription = $_.SMS_BoundaryGroup.Description
		$Script:BoundaryGroupArray += @("$GroupName`t$GroupCode`t$GroupDescription")
	}
}</pre>
That’s it for the functions!  Now, I load all boundaries into a variable so I can refer to it later in the script:
<pre class="toolbar:1 lang:ps decode:true ">$BoundaryObject = Get-WmiObject -Class SMS_Boundary -Namespace $Namespace -ComputerName $Server</pre>
Next, I look for the AD Site on the remote computer. The AD site information is stored in the registry at HKLMSystemCurrentControlSetServicesNetLogonParameters. After I get the AD site, I compare it with all the boundaries that are AD sites. If any are matched, it checks to see if they are in any groups. If they are, the BoundaryGroup function is called. Then, the boundary information is added to an array:
<pre class="toolbar:1 lang:ps decode:true ">foreach ($Boundary in $BoundaryObject) {
	if ($Boundary.BoundaryType -eq 1) {
		$reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $CompName)
		$ADSiteBranch = "SYSTEM\\CurrentControlSet\\services\\Netlogon\\Parameters"
		$ADSiteKey = $reg.OpenSubKey($ADSiteBranch)
		$ADSite = $ADSiteKey.GetValue("DynamicSiteName")
		$BoundaryADSite = $Boundary.Value
		if ($ADSite.ToLower().contains($BoundaryADSite.ToLower())) {
			$BoundaryName = $Boundary.Value
			$BoundaryType = "AD Site"
			$BoundaryDescription = $Boundary.Description
			if ($Boundary.GroupCount -gt 0) {BoundaryGroup $Boundary.BoundaryID}
			$BoundaryArray += @("$BoundaryName`t$BoundaryType`t$BoundaryDescription")
		}
	}
}</pre>
Now, for the IP information. First, I use WMI to go through all the network adapters on the computer, looking for one with an IPv4 address. If there is an IPv4 address, it checks IP range boundaries.  Then it uses the Get-Subnet function to get the subnet and checks the subnet boundaries:
<pre class="toolbar:1 lang:ps decode:true ">$strQuery = "select IPAddress,IpSubnet from Win32_NetworkAdapterConfiguration"
Get-WmiObject -Query $strQuery -Namespace root\cimv2 -ComputerName $CompName | ForEach-Object {
	$ArrayIP = $null
	$ArrayIP = $_.IPAddress
	$ArraySubnet = $_.IpSubnet
	foreach ($instance in $ArrayIP) {
		if ($instance.contains(".")){
			foreach ($Sub in $ArraySubnet) {if ($Sub.contains(".")){$SubnetMask = $Sub}}
			foreach ($Boundary in $BoundaryObject) {
				if ($Boundary.BoundaryType -eq 0) {
					$Subnet = GetSubnet $instance $SubnetMask
					$BoundarySubnet = $Boundary.Value
					if ($BoundarySubnet -eq $Subnet) {
						if ($Boundary.GroupCount -gt 0) {BoundaryGroup $Boundary.BoundaryID}
						$BoundaryName = $Boundary.Value
						$BoundaryType = "AD Site"
						$BoundaryDescription = $Boundary.Description
						$BoundaryArray += @("$BoundaryName`t$BoundaryType`t$BoundaryDescription")
					}
				}
				elseif ($Boundary.BoundaryType -eq 3) {
					$IPRange = $Boundary.Value
					$IPRange = $IPRange.split("-")
					$IpLow = [IPAddress]$IPRange[0]
					$IPHigh = [IPAddress]$IPRange[1]
					$IPToCompare = [IPAddress] $instance
					if ($IPToCompare.Address -ge $IpLow.Address -and $IPToCompare.Address -le $IPHigh.Address) {
						if ($Boundary.GroupCount -gt 0) {BoundaryGroup $Boundary.BoundaryID}
						$BoundaryName = $Boundary.Value
						$BoundaryType = "IP Range"
						$BoundaryDescription = $Boundary.Description
						$BoundaryArray += @("$BoundaryName`t$BoundaryType`t$BoundaryDescription")
					}
				}
			}
		}
	}
}</pre>
That’s it for the scripting!