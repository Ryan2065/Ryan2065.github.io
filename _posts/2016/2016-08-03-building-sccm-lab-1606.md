---
id: 630
title: Building a SCCM Lab with 1606
date: 2016-08-03T15:05:42+00:00
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=630
permalink: /building-sccm-lab-1606/
categories:
  - ConfigMgr
tags:
  - "1606"
  - ConfigMgr
  - Hydration
  - Powershell
  - SCCM
---
One of the things I do pretty often is rebuild my lab. I do this with a series of <a href="https://github.com/Ryan2065/EphingLab" target="_blank">scripts I posted to GitHub called the EphingLab</a>. I decided to post about my SCCM 1511 install script that not only installs SCCM, but also updates it to the latest version (1606 currently).

If you’d just like to get the script and try it out yourselves, the link to <a href="https://github.com/Ryan2065/EphingLab/blob/master/Install%20Scripts/ConfigMgr%201511.ps1" target="_blank">SCCM script is here</a>.

I’m not going to cover the entire script (it’s over 450 lines) but will cover the parts that aren’t part of your standard build scripts.

The first problem when creating this script that I had to solve was an issue of reboots. I can’t very well go and install all the prereqs, SQL, the ADK, and WSUS without needing at least one reboot between them. To solve that issue I came up with a function called AutoLogon and use it with this template:
<pre class="lang:ps decode:true ">Function AutoLogon {
&lt;#
    .SYNOPSIS
        Will run this script again after reboot
 
    .DESCRIPTION
        
   
    .EXAMPLE
        AutoLogon -DefaultUserName 'Home\Administrator' -DefaultPassword 'P@ssw0rd'
  
    .NOTES
        AUTHOR: 
        LASTEDIT: 12/21/2015 22:06:33
 
   .LINK
        https://github.com/Ryan2065/EphingLab
#&gt;
    Param ( $DefaultUserName, $DefaultPassword )
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name AutoAdminLogon -Value 1
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name DefaultUserName -Value "$DefaultUserName"
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name DefaultPassword -Value $DefaultPassword
    $ScriptName = $MyInvocation.ScriptName
    Set-ItemProperty -Path 'HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce' -Name 'EphingScript' -Value "c:\WINDOWS\system32\WindowsPowerShell\v1.0\powershell.exe -noexit -file `"$ScriptName`""
}

$TimesRan = (Get-ItemProperty -Path Registry::HKLM\Software\EphingScripts -ErrorAction SilentlyContinue).TimesRan

If ($TimesRan -eq $null) {
    $TimesRan = 0
    #CODE GOES HERE
    AutoLogon -DefaultUserName $DefaultUserName -DefaultPassword $DefaultPassword
}
elseif ($TimesRan -eq 1) {
    #CODE GOES HERE
    AutoLogon -DefaultUserName $DefaultUserName -DefaultPassword $DefaultPassword
}
elseif ($TimesRan -eq 2) {
    #CODE GOES HERE
    AutoLogon -DefaultUserName $DefaultUserName -DefaultPassword $DefaultPassword
}
$TimesRan++
New-Item -Path Registry::HKLM\Software\EphingScripts -ErrorAction SilentlyContinue
Set-ItemProperty -Path Registry::HKLM\Software\EphingScripts -Name 'TimesRan' -Value $TimesRan
shutdown /r /f /t 0</pre>
The idea is I want to keep running this script, but only want certain sections of code to run depending on the number of times it’s run. So I put a property in the registry with the number of times the script ran, and then keep incrementing the property by 1 every time the script runs.

I then go and install all the required Windows features, and then I set up the System Management container in AD. This code both creates the container and then sets the correct permissions so you don’t have to worry about that:
<pre class="lang:ps decode:true ">$root = (Get-ADRootDSE).defaultNamingContext
$ou = New-ADObject -Type Container -name "System Management" -Path "CN=System,$root" -Passthru 
$acl = Get-ACL "ad:CN=System Management,CN=System,$root"
$computer = Get-ADComputer $env:ComputerName 
$sid = [System.Security.Principal.SecurityIdentifier] $computer.SID
$identity = [System.Security.Principal.IdentityReference] $computer.SID
$adRights = [System.DirectoryServices.ActiveDirectoryRights] "GenericAll"
$type = [System.Security.AccessControl.AccessControlType] "Allow"
$inheritanceType = [System.DirectoryServices.ActiveDirectorySecurityInheritance] "All"
$ace = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $identity,$adRights,$type,$inheritanceType
$acl.AddAccessRule($ace)
Set-ACL -aclobject $acl "ad:CN=System Management,CN=System,$root"</pre>
Then, I reboot and install the ADK, SQL, extend the schema, and install &amp; configure WSUS. Here’s how I’m installing and configuring WSUS:
<pre class="lang:ps decode:true ">Install-WindowsFeature -Name UpdateServices-Services,UpdateServices-DB -IncludeManagementTools
mkdir C:\WSUS
cd 'C:\Program Files\Update Services\Tools\'
.\wsusutil.exe postinstall SQL_INSTANCE_NAME="$env:ComputerName" CONTENT_DIR=C:\WSUS</pre>
I then restart the computer and get onto installing SCCM! After installing SCCM, I enable fast ring updates. A few weeks ago, the ConfigMgr team posted a <a href="https://gallery.technet.microsoft.com/ConfigMgr-1606-Enable-043a8c50" target="_blank">script to TechNet gallery</a> which enables early update ring for SCCM to get production updates before everyone else. I took this script, modified it to work in my lab, and then put the code in my lab build script!
<pre class="lang:ps decode:true ">$WmiObjectSiteClass = "SMS_SCI_SiteDefinition"
$WmiObjectClass = "SMS_SCI_Component"
$WmiComponentName = "ComponentName='SMS_DMP_DOWNLOADER'"
$WmiComponentNameUpdateRing = "UpdateRing" 
$WmiObject = Get-WmiObject -Namespace "root\sms\site_$($SiteCode)" -Class $WmiObjectClass -Filter $WmiComponentName | Where-Object { $_.SiteCode -eq $SiteCode }
#region Enable Fast RIng
$props = $WmiObject.Props
$props = $props | where {$_.PropertyName -eq $WmiComponentNameUpdateRing}
if (!$props) {
    #Create embedded property
    $EmbeddedProperty = ([WMICLASS]"root\SMS\site_$($SiteCode):SMS_EmbeddedProperty").CreateInstance()
    $EmbeddedProperty.PropertyName = $WmiComponentNameUpdateRing
    $EmbeddedProperty.Value = 2
    $EmbeddedProperty.Value1 = ""
    $EmbeddedProperty.Value2 = ""
    $WmiObject.Props += [System.Management.ManagementBaseObject] $EmbeddedProperty
    $WmiObject.put()
}
else
{
    $props = $WmiObject.Props
    $index = 0
    ForEach($oProp in $props)
    {
        if($oProp.PropertyName -eq $WmiComponentNameUpdateRing)
        {
            $oProp.Value=2
            $props[$index]=$oProp;
        }
        $index++
    }

    $WmiObject.Props = $props
    $WmiObject.put()
}</pre>
I then reboot the computer and open up the SCCM console. I use send keys to then open up the PowerShell command prompt through the console and send A (for always) to the scripts. There is a onetime setup piece to the PowerShell cmdlets that imports a certificate. I’m hunting down the code to do this without send keys, but for now this code works!
<pre class="lang:ps decode:true ">Start-Process "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\Microsoft.ConfigurationManagement.exe"
[void] [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic')
[void] [System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
Start-Sleep 10
[System.Windows.Forms.SendKeys]::SendWait('{Enter}')
Start-Sleep 5
[System.Windows.Forms.SendKeys]::SendWait('%F')
Start-Sleep -Milliseconds 50
[System.Windows.Forms.SendKeys]::SendWait('{Down}')
Start-Sleep -Milliseconds 50
[System.Windows.Forms.SendKeys]::SendWait('{Enter}')
Start-Sleep 5
[System.Windows.Forms.SendKeys]::SendWait('A')
Start-Sleep -Milliseconds 50
[System.Windows.Forms.SendKeys]::SendWait('{Enter}')
Start-Sleep 5
AutoLogon -DefaultUserName $DefaultUserName -DefaultPassword $DefaultPassword</pre>
I then reboot (to make it see the certificate and the SCCM PS drive) and start configuring the site! I have code here to install all the standard roles I use, set up boundaries, and set up discover methods. This way the site’s ready and I don’t have these manual steps!
<pre class="lang:ps decode:true ">Import-Module 'C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1'
cd "$($SiteCode):\"

Add-CMApplicationCatalogWebServicePoint -WebsiteName 'Default Web Site' -WebApplicationName 'CMApplicationCatalogSvc' -PortNumber '80' -SiteSystemServerName $ConfigMgrServer -SiteCode $SiteCode -CommunicationType Http
Add-CMApplicationCatalogWebsitePoint -ApplicationWebServicePointServerName $ConfigMgrServer -ConfiguredAsHttpConnection -SiteCode $SiteCode -SiteSystemServerName $ConfigMgrServer
Add-CMAssetIntelligenceSynchronizationPoint -SiteSystemServerName $ConfigMgrServer
Add-CMEndpointProtectionPoint -LicenseAgreed $true -ProtectionService AdvancedMembership -SiteCode $SiteCode -SiteSystemServerName $ConfigMgrServer
Add-CMSoftwareUpdatePoint -SiteCode $SiteCode -SiteSystemServerName $ConfigMgrServer -WsusIisPort 8530 -WsusIisSslPort 8531 -ClientConnectionType Intranet -WsusSsl $false -UseProxy $false
New-CMAccount -UserName 'Home\Administrator' -Password (ConvertTo-SecureString -String 'P@ssw0rd' -AsPlainText -Force)
Add-CMReportingServicePoint -ReportServerInstance "MSSQLSERVER" -DatabaseName "CM_$($SiteCode)" -DatabaseServerName $ConfigMgrServer -SiteCode $SiteCode -SiteSystemServerName $ConfigMgrServer -UserName 'Home\Administrator'

New-CMBoundary -Name 'All IPs' -Type IPRange -Value '192.168.0.1-192.168.255.255'
New-CMBoundaryGroup -Name 'All Devices' -DefaultSiteCode 'PS1'
Add-CMBoundaryToGroup -BoundaryName 'All IPs' -BoundaryGroupName 'All Devices'
Get-CMDistributionPoint | Set-CMDistributionPoint -AddBoundaryGroupName 'All Devices'

Set-CMDiscoveryMethod -ActiveDirectorySystemDiscovery -SiteCode PS1 -Enabled $true
Set-CMDiscoveryMethod -ActiveDirectoryUserDiscovery -SiteCode PS1 -Enabled $true
$Sysdiscovery = Get-CimInstance -Namespace 'root\sms\site_ps1' -classname SMS_SCI_Component -filter 'componentname ="sms_ad_system_discovery_agent"'
$ADContainerProp =$Sysdiscovery.PropLists | where {$_.PropertyListName -eq "AD Containers" }
$ADContainerProp.Values = "LDAP://DC=Home,DC=Lab",0,0
Get-CimInstance -Namespace 'root\sms\site_ps1' -classname SMS_SCI_Component -filter 'componentname ="sms_ad_system_discovery_agent"' | Set-CimInstance -Property @{PropLists=$Sysdiscovery.PropLists}
Invoke-CMSystemDiscovery -SiteCode PS1

$Discovery = Get-CimInstance -Namespace 'root\sms\site_ps1' -classname SMS_SCI_Component -filter 'componentname ="SMS_AD_USER_DISCOVERY_AGENT"'
$ADContainerProp =$Sysdiscovery.PropLists | where {$_.PropertyListName -eq "AD Containers" }
$ADContainerProp.Values = "LDAP://DC=Home,DC=Lab",0,0
Get-CimInstance -Namespace 'root\sms\site_ps1' -classname SMS_SCI_Component -filter 'componentname ="SMS_AD_USER_DISCOVERY_AGENT"' | Set-CimInstance -Property @{PropLists=$Sysdiscovery.PropLists}
</pre>
Lastly, I wait for the site to see an update from Microsoft, and run the first one it sees! Right now, that’s 1606. In the future, the newest update might not be the first, so I’ll need to figure out a better long-term solution for this lab build script. But, for now, this works to start installing 1606!
<pre class="lang:ps decode:true ">$Update = $null
do {
    $ParamHash = @{
        NameSpace = "root\sms\site_$($SiteCode)"
        Query = "Select * from SMS_CM_UpdatePackages where Name like '$($LatestUpdateName)' AND State = 262146"
    }
    $Update = Get-WmiObject @ParamHash
    if($update -eq $null) {
        Write-Host 'No Updates Found'
        $Update
    }
    else {
        Write-Host 'Update found!'
        $Update
    }
    Start-Sleep 5
} while ($Update -eq $null)

$ParamHash = @{
    NameSpace = "root\sms\site_$($SiteCode)"
    Query = "Select * from SMS_CM_UpdatePackages where Name like '$($LatestUpdateName)'"
}
$Update = Get-WmiObject @ParamHash
$Update.UpdatePrereqAndStateFlags(0,2)</pre>
And then that’s it! The script is still a work in progress. I realized I didn’t create a client push account during this process, so I’ll come back and add that to my configuration steps. But this should be a good starting point if you want to not only install SCCM, but configure it automatically whenever you rebuild your lab!

*Edit 8/10/2016*

I added 'AND State = 262146' to the end of the query in the code to start the 1606 update. Thank you to Richard Xia in the comments for pointing out the script needed to wait for the update to go to that state before initiating the install.