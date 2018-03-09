---
id: 254
title: 'A ConfigMgr Admin&#8217;s Guide to WOL Magic Packet: Part Two'
date: 2013-12-15T19:15:07+00:00
author: Ryan Ephgrave
layout: post
guid: https://www.ephingadmin.com/?p=254
permalink: /a-configmgr-admins-guide-to-wol-magic-part-two/
categories:
  - ConfigMgr
  - WOL
---
In part one of this guide, I walked you through the steps you’ll need to follow to enable WOL Magic Packet on your site server.  Now I will go through client settings you may need to change to get this working on individual computers.

First off, I would highly recommend you have standardized drivers and BIOS versions. Throughout the years, I’ve found that WOL Magic Packet tends to be a setting that’s very common for manufacturers to screw up in new driver and BIOS releases. Whenever you put a new BIOS version or NIC driver out in your environment, make sure you test to make sure you can wake the computer from a powered off state, sleep mode, and a powered off state after a power loss.

I’m not going to be able to provide you screen shots for all the different BIOS versions out there, since it changes by version number, manufacturer, and model. Generally the WOL settings are found under the power settings, so make sure WOL is enabled there.

On to the Windows settings! Go to Control Panel, Network and Sharing Center, Change Adapter Settings (On the left), Right click on your Ethernet device, click properties, click configure, and here you should see the Power Management tab.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image7.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image7" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image7_thumb.png" alt="image7" width="326" height="225" border="0" /></a>

These boxes (theoretically) will only affect the computer during sleep mode. When a computer is powered off, Windows cannot do anything in terms of turning the computer back on. Your BIOS settings are what affect the computer when it is powered off. The third box is up to you if you want to check it or not. Magic packet will work to wake up the computer, but so will other technologies. If you only want the computer to wake up the computer with Magic Packet, check the third box.

Now, go to Advanced, and look for Wake On Magic Packet. Make sure this is enabled.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image8.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image8" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image8_thumb.png" alt="image8" width="261" height="295" border="0" /></a>

If you are using an Intel NIC, there MAY be another setting there called Wake on Magic Packet from power off state. This could also show up on the Power Management tab. Enable this setting also to make sure you can wake up the computer. This particular setting affects the computer when it is powered off, not sleep mode. Windows can put the NIC in a low power mode when the computer is shut down, and if it does you won’t be able to wake up the computer. From what I’ve seen, most Intel NICs have this setting, but you don’t always see the option in the properties of the NIC. If you don’t see it, you will need to enable it in the registry. More on that below…

Now, for the two settings Microsoft really hides on us. If a computer is in sleep mode, and you wake it with a magic packet, Windows will turn the computer back off after 2 minutes. This doesn’t really give ConfigMgr any time to do anything, so you will want to change that to something more reasonable. I usually change it to 20 minutes. The registry key is located here: HKEY_LOCAL_MACHINESYSTEMCurrentControlSetControlPowerPowerSettings238C9FA8-0AAD-41ED-83F4-97BE242C8F207bc4a2f9-d8fc-4469-b07b-33eb785aaca0 and you can see the description of this key is “Idle timeout before the system returns to a low power sleep state after waking unattended.”  If you are following along in the registry, expand DefaultPowerSchemeValues, and you’ll see a few GUIDs. Each GUID is a different Power Plan, so by default there will be three. You only need to change the setting for your current Power Plan for it to take effect.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image9.png"><img class="alignnone" style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image9" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image9_thumb.png" alt="image9" width="298" height="153" border="0" /></a>

There is an ACSettingIndex and DCSettingIndex, which are for plugged in and battery. The decimal version of these keys shows you how many seconds the computer will wait before going back to sleep. The default is 120 seconds. I generally just add a 0 to make it 20 minutes, or 1200 seconds. Change it for ACSettingIndex for plugged in, and the DCSettingIndex if you want to change the on battery value.

If you want, you can go to all the computers in your environment and change these settings manually, or you can use a script to do it! The more I use compliance settings, the more I like them, so I’ll walk you through how to make a compliance setting for this.

Go to Assets and Compliance in the ConfigMrg console, expand Compliance Settings, right click on Configuration Items, and click Create Configuration Item.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image10.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image10" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image10_thumb.png" alt="image10" width="263" height="281" border="0" /></a>

On this first screen, just fill out a descriptive name and if you want, put something in the Description.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image11.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image11" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image11_thumb.png" alt="image11" width="336" height="296" border="0" /></a>

On the second screen choose what OS you want this to run on. I don’t know if this will work on Windows XP or Server 2003 or Windows Embedded, so I won’t enable it for those OS.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image12.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image12" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image12_thumb.png" alt="image12" width="367" height="325" border="0" /></a>

Now, the settings screen. Click New and give this a name. The first setting I’m going to set is the one that allows the computer to wake from a sleep state. The setting time should be Script, and Data Type should be Boolean.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image13.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image13" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image13_thumb.png" alt="image13" width="337" height="316" border="0" /></a>

Now, click on Add Script under discovery script, make sure the dropdown is Windows PowerShell, and add this script:

 
<pre class="lang:ps decode:true " >$PNPCapabilities = 256
$Index = $null
$query = "select * from Win32_NetworkAdapterConfiguration Where IPEnabled = 'True'"
$PNPCapabilitiesSet = $true
Get-WmiObject -Query $query | ForEach-Object {
    $Index = $_.Index
	if ($Index.Length -ne 4) {
		do {
			$Index = "0$Index"
		} while ($Index.Length -lt 4)
	}
	$RegKey = Get-ItemProperty -Path "Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\$Index"
	If ($RegKey.pnpcapabilities -ne $PNPCapabilities) {
    	$PNPCapabilitiesSet = $false
	}
}
$PNPCapabilitiesSet</pre> 


The first line, $PNPCapabilities = 256 will check all three boxes on the Power Management tab:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image14.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image14" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image14_thumb.png" alt="image14" width="244" height="99" border="0" /></a>

If you just want the first two boxes checked, set it to 0.

Now, the next series of lines does a WMI query to get the current NIC. It then makes sure the length of the index number is 4 characters. If it isn’t (and it usually isn’t) it will add zeroes to the front of the number until it is 4 characters. Now, it goes to the registry here: HKEY_LOCAL_MACHINESYSTEMCurrentControlSetControlClass{4d36e972-e325-11ce-bfc1-08002be10318} and then uses the index number to find the correct NIC key. The PNPCapabilities property handles all three of those check boxes, so it sets it to either 256 or 0, depending on which boxes you want checked.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image15.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image15" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image15_thumb.png" alt="image15" width="241" height="244" border="0" /></a>

Click Ok and now click Add Script… under Remediation script (optional), change the script language to Windows Powershell, and add this script:

 
<pre class="lang:ps decode:true " >$PNPCapabilities = 256
$Index = $null
$query = "select * from Win32_NetworkAdapterConfiguration Where IPEnabled = 'True'"
Get-WmiObject -Query $query | ForEach-Object {
    $Index = $_.Index
	if ($Index.Length -ne 4) {
		do {
			$Index = "0$Index"
		} while ($Index.Length -lt 4)
	}
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\$Index" -Name "PNPCapabilities" -Value "$PNPCapabilities" -Type "DWORD" -Force
}</pre> 


Set PNPCapabilities to the same number you set in the detection script, and then click Ok. This script changes the registry key to the correct setting. Now, click on the Compliance Rules tab, and click New. We have made the detection script, and now we are going to tell ConfigMgr what to do with that detection script.  Name the rule whatever you want, Rule Type should be Value, and “the following values:” should be True. If you want to remediate (change the setting to the correct setting) then check the box. This will run the remediation script I had you create earlier.
<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image16.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image16" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image16_thumb.png" alt="image16" width="283" height="290" border="0" /></a>

Click Ok, and Ok, and now you are back to the Settings screen. We only created scripts for one of four settings, so now create settings for Wake on Magic Packet with these scripts:

Detection:

<pre class="lang:ps decode:true " >$Index = $null
$query = "select * from Win32_NetworkAdapterConfiguration Where IPEnabled = 'True'"
$WakeOnPacketSet = $true
Get-WmiObject -Query $query | ForEach-Object {
    $Index = $_.Index
	if ($Index.Length -ne 4) {
		do {
			$Index = "0$Index"
		} while ($Index.Length -lt 4)
	}
	$RegKey = Get-ItemProperty -Path "Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\$Index"
	If ($RegKey."*WakeOnMagicPacket" -ne 1) {
    	$WakeOnPacketSet = $false
	}
}
$WakeOnPacketSet</pre> 

Remediation:

<pre class="lang:ps decode:true " >$Index = $null
$query = "select * from Win32_NetworkAdapterConfiguration Where IPEnabled = 'True'"
Get-WmiObject -Query $query | ForEach-Object {
    $Index = $_.Index
	if ($Index.Length -ne 4) {
		do {
			$Index = "0$Index"
		} while ($Index.Length -lt 4)
	}
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\$Index" -Name "*WakeOnMagicPacket" -Value "1" -Type "String" -Force
}</pre> 


The property you want to change here is *WakeOnMagicPacket, and it needs to be 1 for enabled.

To change the “Idle timeout before the system returns to a low power sleep state after waking unattended” plugged in value, here is the detection script:

 
<pre class="lang:ps decode:true " >try {
    $SecondsToWait = 1200
    $ACSettingSet = $true
    $strQuery = "Select InstanceID from Win32_PowerPlan where IsActive = 'true'"
    $PowerPlanInstanceID = (Get-WmiObject -Query $strQuery -Namespace root\cimv2\power).InstanceID.ToString()
    $RegEx = [RegEx]"{(.*?)}$"
    $GUID = $RegEx.Match($PowerPlanInstanceID).Groups[1].Value
    $RegKeys = Get-ItemProperty -Path "Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0\DefaultPowerSchemeValues\$GUID"
    If ($RegKeys.AcSettingIndex -ne $SecondsToWait) {
        $ACSettingSet = $false
    }
    return $ACSettingSet
}
catch {
    return $false
}</pre> 


And here is the remediation script:

 
<pre class="lang:ps decode:true " >$ErrorActionPreference = "SilentlyContinue"
$SecondsToWait = 1200
$strQuery = "Select InstanceID from Win32_PowerPlan where IsActive = 'true'"
$PowerPlanInstanceID = (Get-WmiObject -Query $strQuery -Namespace root\cimv2\power).InstanceID.ToString()
$RegEx = [RegEx]"{(.*?)}$"
$GUID = $RegEx.Match($PowerPlanInstanceID).Groups[1].Value
New-Item -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0\DefaultPowerSchemeValues\$GUID"
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0\DefaultPowerSchemeValues\$GUID" -Name "AcSettingIndex" -Value "$SecondsToWait" -Type "DWORD" -Force</pre> 


The script finds the current power plan through WMI, uses regex to get the GUID, and then changes the correct setting.

Lastly, the setting for Wake on Magic Packet from power off state. I believe this setting is Intel specific, so if you don’t want to roll this setting into the Windows Compliance Items,  you will need to make a new Compliance Item for your Intel NIC computers. The property here is called EnablePME and it needs to be set to 1 for it to be enabled.

Here is the detection script:

 
<pre class="lang:ps decode:true " >$Index = $null
$query = "select * from Win32_NetworkAdapterConfiguration Where IPEnabled = 'True'"
$EnablePMESet = $true
Get-WmiObject -Query $query | ForEach-Object {
    $Index = $_.Index
	if ($Index.Length -ne 4) {
		do {
			$Index = "0$Index"
		} while ($Index.Length -lt 4)
	}
	$RegKey = Get-ItemProperty -Path "Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\$Index"
	If ($RegKey.EnablePME -ne 1) {
    	$EnablePMESet = $false
	}
}
$EnablePMESet</pre> 


And here is the remediation script:

 
<pre class="lang:ps decode:true " >$Index = $null
$query = "select * from Win32_NetworkAdapterConfiguration Where IPEnabled = 'True'"
Get-WmiObject -Query $query | ForEach-Object {
    $Index = $_.Index
	if ($Index.Length -ne 4) {
		do {
			$Index = "0$Index"
		} while ($Index.Length -lt 4)
	}
    Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\$Index" -Name "EnablePME" -Value "1" -Type "String" -Force
}</pre> 


Once you are done, your screen should look something like this:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image17.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image17" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image17_thumb.png" alt="image17" width="337" height="287" border="0" /></a>

Click Next, and your screen should now look like this:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image18.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image18" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image18_thumb.png" alt="image18" width="334" height="289" border="0" /></a>

There should be one Compliance Rule for each Compliance Setting. You set these in the Compliance Setting in that second tab:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image19.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image19" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image19_thumb.png" alt="image19" width="385" height="85" border="0" /></a>

If you don’t have one per setting, you will need to create one for the missing setting. To me, it’s easier to just go back to settings, edit the one that’s missing, and add it there. Make sure you are remediating the ones you want to remediate. You can quickly see which ones are being remediated in the Remediate column.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image20.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image20" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image20_thumb.png" alt="image20" width="102" height="120" border="0" /></a>

Click Next, and Next to create this.

Now that it is created, you will need to create a configuration baseline to deploy.  Right click on Configuration Baselines, select Create Configuration Baseline:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image21.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image21" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image21_thumb.png" alt="image21" width="244" height="86" border="0" /></a>

Name this something descriptive and then click Add, Configuration Items

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image22.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image22" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image22_thumb.png" alt="image22" width="287" height="260" border="0" /></a>

Select the configuration item you created, and then click Add

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image23.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image23" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image23_thumb.png" alt="image23" width="333" height="323" border="0" /></a>

Click Ok, and then Ok to create it. Right click on the new baseline and select Deploy. Now you are going to see a very different Deploy screen than you normally see.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image24.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image24" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image24_thumb.png" alt="image24" width="332" height="335" border="0" /></a>

On here, check Remediate noncompliance rules when supported if you want to remediate, and check the one below that if you want to remediate outside of maintenance windows. Since you are only changing a few registry keys, it is probably ok to do this outside of a maintenance window. If you want to generate an alert for this, you can set those settings next. Now select your collection, and the schedule you want this deployment to run on. I’m going to set it to run once a week. This way if a power plan is changed, or a network driver is updated you won’t have to worry about going back and changing those settings again.

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/01/image25.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image25" src="http://www.ephingadmin.com/wp-content/uploads/2016/01/image25_thumb.png" alt="image25" width="337" height="338" border="0" /></a>

Click Ok and you are done!  The configuration baseline is all set.

In the next part, I’ll go through some troubleshooting steps you can go through for those computers that don’t wake up.