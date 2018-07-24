---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1559
Published Date: 
Archived Date: 2009-12-26t18
---

# backup all esxi - 

## Description

back up all your esxi hosts to a local directory.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $backupDir = "c:\backups"
 
 $esxiHosts = Get-VMHost | Where { $_ | Get-View -Property Config | Where { $_.Config.Product.ProductLineId -eq "embeddedEsx" } }
 
 $esxiHosts | Foreach {
 	$fullPath = $backupDir + "\" + $_.Name
 	mkdir $fullPath -ea SilentlyContinue | Out-Null
 	Set-VMHostFirmware -VMHost $_ -BackupConfiguration -DestinationPath $fullPath
 }
`

