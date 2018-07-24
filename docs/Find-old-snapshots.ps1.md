---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5737
Published Date: 2015-02-15t19
Archived Date: 2015-10-16t20
---

# find old snapshots - 

## Description

simple powercli example to find old snapshots. note that i would not actually use the techniques shown here, this script was intentionally written this way as a part of a training video in which i am building on techniques.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param ( $Age = 30 )
 
 Connect-VIServer vcenter.domain.com
 $vm = Get-VM
 $snapshots = Get-Snapshot -VM $vm
 Write-Host -ForegroundColor Red "Old snapshots found:"
 foreach ( $snap in $snapshots ) {
 	if ( $snap.Created -lt (Get-Date).AddDays( -$Age ) ) {
 		Write-Host "Name: " $snap.Name "  Size: " $snap.SizeMB "  Created: " $snap.Created
 	}
 }
`

