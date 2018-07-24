---
Author: lunahex
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6227
Published Date: 2016-02-19t21
Archived Date: 2016-08-26t03
---

#  - 

## Description

kills trustedinstaller

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function killItWithFire {
   $TrustedInstallerProcess = Get-Process -ProcessName trustedinstaller
   if($TrustedInstallerProcess){
     Write-Host -ForegroundColor GREEN "TrustedInstaller process is running, killing it now."
     Stop-Process $TrustedInstallerProcess.Id -Force
   } else {
     Write-Host "TrustedInstaller process is not running."
   }
   $TrustedInstallerService = Get-Service trustedinstaller
   if($TrustedInstallerService.status -eq "Running"){
     Write-Host -ForegroundColor GREEN "TrustedInstaller service is running, stopping it now."
     Set-Service $TrustedInstallerService.name -Status stopped
   }
   Write-Host -ForegroundColor GREEN "Disabling TrustedInstaller service."
   Set-Service $TrustedInstallerService.name -StartupType Disabled
 }
 killItWithFire
`

