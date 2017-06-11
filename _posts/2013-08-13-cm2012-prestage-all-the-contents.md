---
id: 98
title: CM2012 – Prestage all the contents!
date: 2013-08-13T15:23:01+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=98
permalink: /cm2012-prestage-all-the-contents/
categories:
  - ConfigMgr
  - Powershell
tags:
  - ConfigMgr
  - Configuration Manager
  - Content
  - Redistribute
---
A co-worker sent me an e-mail with a link to David O’Brien’s <a href="http://www.david-obrien.net/2013/02/12/how-to-prestage-content-in-configuration-manager-2012-script/" target="_blank">post</a> about a script that prestages content in a specific folder.  He asked me if I could modify this script to prestage all the content, not just ones in certain folders. I decided the best course of action was to just write the thing from scratch, since prestaging everything is much simpler than doing certain folders.

In this script, change $SiteCode, $Server, $DP, and $FolderToSaveTo to relevant fields for your organization.
<pre class="lang:ps decode:true">$SiteCode = "SITECODE"
$Server = "SERVER"
$DP = "FQDNOfDP"
$FolderToSaveTo = "C:\SavingHere\"</pre>
The content needs to be on the DP specified in order to prestage. If not, you’ll get a big wall o’ text full of errors.  The script won’t stop, but there will be an intimidating amount of red in the console window. Also, the DP name needs to be what is listed in the Administration tab of the console window:
<p id="sxPBUIY"><img class="alignnone size-full wp-image-99 " src="http://www.ephingadmin.com/wp-content/uploads/2015/11/img_564c976c328c1.png" alt="" /></p>
Here, the DP name would be CM2012.LAB.HOME

If CM2012 console isn’t installed in the default directory, change the module path to the correct path:
<pre class="lang:ps decode:true">Import-Module "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1"
</pre>
Lastly, $Overwrite is only used if the file already exists. This is there if you want to run the script every week and only have it add new content. If you set $Overwrite = $true, then all app content is re-prestaged and the previous content is removed as it is found.

This uses the CM2012 Powershell modules so it will need to be run from a 32-bit Powershell window.
<pre class="lang:ps decode:true  ">if ([Environment]::Is64BitProcess) {Read-Host "This must be run from 32-bit Powershell";exit}
 
$SiteCode = "SITECODE"
$Server = "SERVER"
$DP = "FQDNOfDP"
$FolderToSaveTo = "C:\SavingHere\"
Import-Module "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1"
#$Overwrite = $true
$Overwrite = $false
 
$Namespace = "root\SMS\site_" + $SiteCode
$SiteDrive = $SiteCode + ":\"
if ($FolderToSaveTo.EndsWith("\")) {$FolderToSaveTo = $FolderToSaveTo.SubString(0,$FolderToSaveTo.Length - 1)}
if(!(Test-Path $FolderToSaveTo)) {New-Item -ItemType Directory -Path $FolderToSaveTo}
if(!(Test-Path "$FolderToSaveTo\Applications")) {New-Item -ItemType Directory -Path "$FolderToSaveTo\Applications"}
if(!(Test-Path "$FolderToSaveTo\Packages")) {New-Item -ItemType Directory -Path "$FolderToSaveTo\Packages"}
if(!(Test-Path "$FolderToSaveTo\Driver_Packages")) {New-Item -ItemType Directory -Path "$FolderToSaveTo\Driver_Packages"}
if(!(Test-Path "$FolderToSaveTo\Software_Update_Packages")) {New-Item -ItemType Directory -Path "$FolderToSaveTo\Software_Update_Packages"}
if(!(Test-Path "$FolderToSaveTo\Boot_Image_Packages")) {New-Item -ItemType Directory -Path "$FolderToSaveTo\Boot_Image_Packages"}
if(!(Test-Path "$FolderToSaveTo\OS_Images")) {New-Item -ItemType Directory -Path "$FolderToSaveTo\OS_Images"}
 
Get-WmiObject -Query "Select * from SMS_Application where ISLatest='true'" -ComputerName $Server -Namespace $Namespace | ForEach-Object {
	$AppID = $_.CI_UniqueID
	$AppName = $_.LocalizedDisplayName
	$FileName = "$FolderToSaveTo\Applications\$AppName.pkgx"
	CD "C:\"
	$Skip = $false
	if ($Overwrite -eq $false) {if (Test-Path $FileName) {$Skip = $true}}
	if ($Skip -eq $false) {
		Remove-Item $FileName -Force
		CD "$SiteDrive"
		Publish-CMPrestageContent -ApplicationName $AppName -DisableExportAllDependencies -FileName $FileName -DistributionPointName $DP
	}
}
 
Get-WmiObject -Query "Select * from SMS_Package" -ComputerName $Server -Namespace $Namespace | ForEach-Object {
	$PackageName = $_.Name
	$FileName = "$FolderToSaveTo\Packages\$PackageName.pkgx"
	CD "C:\"
	$Skip = $false
	if ($Overwrite -eq $false) {if (Test-Path $FileName) {$Skip = $true}}
	if ($Skip -eq $false) {
		Remove-Item $FileName -Force
		CD "$SiteDrive"
		Publish-CMPrestageContent -PackageName $PackageName -FileName $FileName -DistributionPointName $DP
	}
}
 
Get-WmiObject -Query "Select * from SMS_DriverPackage" -ComputerName $Server -Namespace $Namespace | ForEach-Object {
	$PackageName = $_.Name
	$FileName = "$FolderToSaveTo\Driver_Packages\$PackageName.pkgx"
	CD "C:\"
	$Skip = $false
	if ($Overwrite -eq $false) {if (Test-Path $FileName) {$Skip = $true}}
	if ($Skip -eq $false) {
		Remove-Item $FileName -Force
		CD "$SiteDrive"
		Publish-CMPrestageContent -DriverPackageName $PackageName -FileName $FileName -DistributionPointName $DP
	}
}
 
Get-WmiObject -Query "SELECT * FROM SMS_SoftwareUpdatesPackage WHERE ActionInProgress!=3" -ComputerName $Server -Namespace $Namespace | ForEach-Object {
	$PackageName = $_.Name
	$FileName = "$FolderToSaveTo\Software_Update_Packages\$PackageName.pkgx"
	CD "C:\"
	$Skip = $false
	if ($Overwrite -eq $false) {if (Test-Path $FileName) {$Skip = $true}}
	if ($Skip -eq $false) {
		Remove-Item $FileName -Force
		CD "$SiteDrive"
		Publish-CMPrestageContent -DeploymentPackageName $PackageName -FileName $FileName -DistributionPointName $DP
	}
}
 
Get-WmiObject -Query "SELECT * FROM SMS_BootImagePackage WHERE ActionInProgress!=3" -ComputerName $Server -Namespace $Namespace | ForEach-Object {
	$PackageName = $_.Name
	$FileName = "$FolderToSaveTo\Boot_Image_Packages\$PackageName.pkgx"
	CD "C:\"
	$Skip = $false
	if ($Overwrite -eq $false) {if (Test-Path $FileName) {$Skip = $true}}
	if ($Skip -eq $false) {
		Remove-Item $FileName -Force
		CD "$SiteDrive"
		Publish-CMPrestageContent -BootImageName $PackageName -FileName $FileName -DistributionPointName $DP
	}
}
 
Get-WmiObject -Query "SELECT * FROM SMS_ImagePackage WHERE ActionInProgress!=3" -ComputerName $Server -Namespace $Namespace | ForEach-Object {
	$PackageName = $_.Name
	$FileName = "$FolderToSaveTo\OS_Images\$PackageName.pkgx"
	CD "C:\"
	$Skip = $false
	if ($Overwrite -eq $false) {if (Test-Path $FileName) {$Skip = $true}}
	if ($Skip -eq $false) {
		Remove-Item $FileName -Force
		CD "$SiteDrive"
		Publish-CMPrestageContent -OperatingSystemImageName $PackageName -FileName $FileName -DistributionPointName $DP
	}
}</pre>