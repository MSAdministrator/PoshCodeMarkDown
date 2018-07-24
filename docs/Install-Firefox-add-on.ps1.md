---
Author: scott copus
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4919
Published Date: 2015-02-20t21
Archived Date: 2015-05-08t17
---

# install firefox add-on - 

## Description

powershell script to installs firefox add-ons (extensions)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 #
 
 
 $FFinstallDir = "${Env:ProgramFiles(x86)}\Mozilla Firefox"
 $FFextensionsRegKey = "HKLM:\SOFTWARE\Wow6432Node\Mozilla\Firefox\Extensions"
 if (!(Test-Path -Path $FFinstalldir)) {
 	$FFinstallDir    = "${Env:ProgramFiles}\Mozilla Firefox"
 	$FFextensionsRegKey = "HKLM:\SOFTWARE\Mozilla\Firefox\Extensions"
 }
 if (!(Test-Path -Path $FFinstallDir)) {
 	Write-Host "Problem: Could not find Firefox installation folder."
 }
 
 $FFextensionsDir = "$FFinstallDir\extensions"
 if (!(Test-Path -Path $FFextensionsDir)) {
     New-Item $FFextensionsDir -ItemType Directory -ErrorAction SilentlyContinue -Force | Out-Null
 }
 if (!(Test-Path -Path $FFextensionsRegKey)) {
     New-Item $FFextensionsRegKey -ErrorAction SilentlyContinue -Force | Out-Null
 }
 
 Get-ChildItem . -Directory | ForEach-Object {
 	write-host Installing Firefox extension: $_.Name
 	Copy-Item $_.FullName $FFextensionsDir -Recurse -Force | Out-Null
 	New-ItemProperty -Path $FFextensionsRegKey -Name $_.Name -Value "$FFextensionsDir\$($_.Name)" -PropertyType String -Force | Out-Null
 }
`

