---
Author: kenneth c mazie
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2567
Published Date: 2012-03-16t10
Archived Date: 2017-02-10t15
---

# wsus-settings.ps1 - 

## Description

as written will manually apply all settings associated with a local wsus server.  ideal for use when you need to force a non-domain system to point to a domain based wsus server.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #==================================================================================================
 #
 #
 #==================================================================================================
 
 Clear-Host
 
 #--[ Windows Update Server Info ]--
 $TargetGroup = "Computers"
 $WUServer = "http://192.168.1.10"
 $WUStatusServer = "http://192.168.1.10"
 #-[ NOTE: all other settings should be set below ]--
 
 #--[ Setup Windows Updates ]--
 Write-Host -backgroundColor white -foregroundcolor blue -object "Setting WSUS Parameters..."
 
 if(!( Test-Path 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' ))
 {
       New-Item 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -force
 }
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -name 'ElevateNonAdmins' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -name 'AcceptTrustedPublisherCerts' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -name 'TargetGroupEnabled' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -name 'TargetGroup' -value $TargetGroup -propertyType "String" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -name 'WUServer' -value $WUServer -propertyType "String" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate' -name 'WUStatusServer' -value $WUStatusServer -propertyType "String" -force
 
 if(!( Test-Path 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' ))
 {
       New-Item 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -force
 }
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'NoAutoRebootWithLoggedOnUsers' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'NoAUShutdownOption' -value '0' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'NoAUAsDefaultShutdownOption' -value '0' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'DetectionFrequencyEnabled' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'DetectionFrequency' -value '22' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'AutoInstallMinorUpdates' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'RebootWarningTimeoutEnabled' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'RebootWarningTimeout' -value '5' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'RebootRelaunchTimeoutEnabled' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'RebootRelaunchTimeout' -value '30' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'IncludeRecommendedUpdates' -value '22' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'AUPowerManagement' -value '0' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'NoAutoUpdate' -value '0' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'AUOptions' -value '4' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'ScheduledInstallDay' -value '4' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'ScheduledInstallTime' -value '3' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'UseWUServer' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'RescheduleWaitTimeEnabled' -value '1' -propertyType "DWord" -force
 New-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\WindowsUpdate\AU' -name 'RescheduleWaitTime' -value '1' -propertyType "DWord" -force
 
 Write-Host -backgroundColor white -foregroundcolor blue -object "Completed..."
`

