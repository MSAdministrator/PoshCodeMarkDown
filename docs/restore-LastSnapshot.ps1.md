---
Author: txguy
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6159
Published Date: 2016-12-31t19
Archived Date: 2016-03-19t02
---

# restore-lastsnapshot - 

## Description

revert to the last vmware snapshot with variables for vvcenter and vm

## Comments



## Usage

restore-lastsnapshot -vcenter vcenter1 -vm vm1234

## TODO



## 

``

## Code

`#
 #
 #
 #
 #
 
 
 param(
 	[string]$vcenter,
 	[string]$vm
 )
 
 
 connect-viserver -server $vcenter
 $snap = Get-Snapshot -VM $vm | Sort-Object -Property Created -Descending | Select -First 1
 Set-VM -VM $vm -SnapShot $snap -Confirm:$false
 start-vm -vm $vm
`

