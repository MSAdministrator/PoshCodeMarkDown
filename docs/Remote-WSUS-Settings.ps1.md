---
Author: paul brice
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1343
Published Date: 2010-09-25t10
Archived Date: 2017-02-10t15
---

# remote wsus settings - 

## Description

powershell script to examine remote wsus configuration for multiple servers. this code places the registry strings into a custom object which results in the data from multiple server being very accessible.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $WSUSREGALL = @()
 [String]$File = "C:\server.txt"
 $Servers = Get-Content $File
 ForEach($Server In $Servers)
 {
 $Registry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $Server)
 $RegKey0 = $Registry.OpenSubKey("Software\policies\Microsoft\Windows\WindowsUpdate\" )
 $RegKey1 = $Registry.OpenSubKey("Software\policies\Microsoft\Windows\WindowsUpdate\AU\")
 $WSUSREG = New-Object System.Object
 $WSUSREG | Add-Member -MemberType NoteProperty -Name "Server" -Value $Server
 $WSUSREG | Add-Member -MemberType NoteProperty -Name WUServer -Value $RegKey0.GetValue("WUServer")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name WUStatusServer -Value $RegKey0.GetValue("WUStatusServer")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name TargetGroupEnabled -Value $RegKey0.GetValue("TargetGroupEnabled")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name AcceptTrustedPublisherCerts -Value $RegKey0.GetValue("AcceptTrustedPublisherCerts")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name AUOptions -Value $RegKey1.GetValue("AUOptions")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name NoAutoUpdate -Value $RegKey1.GetValue("NoAutoUpdate")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name NoAUShutdownOption -Value $RegKey1.GetValue("NoAUShutdownOption")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name NoAUAsDefaultShutdownOption -Value $RegKey1.GetValue("NoAUAsDefaultShutdownOption")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name ScheduledInstallDay -Value $RegKey1.GetValue("ScheduledInstallDay")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name ScheduledInstallTime -Value $RegKey1.GetValue("ScheduledInstallTime")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name UseWUServer -Value $RegKey1.GetValue("UseWUServer")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name RescheduleWaitTimeEnabled -Value $RegKey1.GetValue("RescheduleWaitTimeEnabled")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name RescheduleWaitTime -Value $RegKey1.GetValue("RescheduleWaitTime")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name NoAutoRebootWithLoggedOnUsers -Value $RegKey1.GetValue("NoAutoRebootWithLoggedOnUsers")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name DetectionFrequencyEnabled -Value $RegKey1.GetValue("DetectionFrequencyEnabled")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name DetectionFrequency -Value $RegKey1.GetValue("DetectionFrequency")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name AutoInstallMinorUpdates -Value $RegKey1.GetValue("AutoInstallMinorUpdates")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name RebootWarningTimeoutEnabled -Value $RegKey1.GetValue("RebootWarningTimeoutEnabled")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name RebootWarningTimeout -Value $RegKey1.GetValue("RebootWarningTimeout")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name RebootRelaunchTimeoutEnabled -Value $RegKey1.GetValue("RebootRelaunchTimeoutEnabled")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name RebootRelaunchTimeout -Value $RegKey1.GetValue("RebootRelaunchTimeout")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name IncludeRecommendedUpdates -Value $RegKey1.GetValue("IncludeRecommendedUpdates")
 $WSUSREG | Add-Member -MemberType NoteProperty -Name AUPowerManagement -Value $RegKey1.GetValue("AUPowerManagement")
 $WSUSREGALL += $WSUSREG
 }
 $WSUSREGALL
 $WSUSREGALL | Export-Csv "C:\DataWSUS.csv" -NoTypeInformation
`

