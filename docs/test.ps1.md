---
Author: teste
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6452
Published Date: 2016-07-21t15
Archived Date: 2016-07-27t03
---

# test - 

## Description

test

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
     Get-VM
     $password = ConvertTo-SecureString -AsPlainText -Force -String 'Pa$$w0rd'
 
     $iso_X = 'C:\depot\iso_x\en_windows_server_2016_technical_preview_4_x64_dvd_7258292\'
     $VirtualHardDiskPath = (Get-VMHost).VirtualHardDiskPath 
 
     Import-Module "C:\NanoServer_TP4\NanoServerImageGenerator.psm1 "
 
 
 
 
     $nanoName = 'sea-nano11'
     $nanoHardDisk = "$VirtualHardDiskPath\$nanoName.vhd"
 
     if (Test-Path $nanoHardDisk) {
       Get-ChildItem $nanoHardDisk | Remove-Item -Confirm
     }
`

