---
Author: cody bunch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1953
Published Date: 2010-07-07t11
Archived Date: 2013-11-15t09
---

# roll-dice.ps1 - 

## Description

a really bad roll-dice script to do ‘bad things’ to vmware snapshots taken on the pipeline.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Begin {
 	$rand = New-Object System.Random
 	$dice = $rand.next(1,4)
 }
 
 Process {
 	if ( $_ -isnot [VMware.VimAutomation.Types.Snapshot] ) { continue }
 	if ($dice -gt 1) { 
 		$_ | Remove-Snapshot -Confirm:$false
 		Write-Host "OH NOES! Snapshot $_ Has been deleted!`n" 
 	} else {
 		Write-Host "Snapshot $_ lives to fight again!`n"
 	}
 }
`

