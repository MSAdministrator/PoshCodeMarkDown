---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1346
Published Date: 
Archived Date: 2009-10-25t12
---

# nic performance - 

## Description

reads perfmon counters from all network interfaces

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $cat = New-Object system.Diagnostics.PerformanceCounterCategory("Network Interface")
 $inst = $cat.GetInstanceNames()
 foreach ( $nic in $inst ) {
 	$a = $cat.GetCounters( $nic )
 	$a | ft CounterName, { $_.NextValue() } -AutoSize
 }
`

