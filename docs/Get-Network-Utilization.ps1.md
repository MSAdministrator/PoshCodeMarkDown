---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6510
Published Date: 2016-09-12t09
Archived Date: 2016-10-29t07
---

# get network utilization - 

## Description

get utilization from all network interfaces

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $counters = @()
 foreach ($inst in (new-object System.Diagnostics.PerformanceCounterCategory("network interface")).GetInstanceNames()){
 	$cur = New-Object system.Diagnostics.PerformanceCounter('Network Interface','Bytes Total/sec',   $inst)
 	$max = New-Object system.Diagnostics.PerformanceCounter('Network Interface','Current Bandwidth', $inst)
 
 	$cur.NextValue() | Out-Null
 	$max.NextValue() | Out-Null
 
 	$counters += @{"Throughput"=$cur;"Bandwidth"=$max;"Name"=$inst}
 }
 
 sleep 2
 
 foreach($counter in $counters) {
 
 	$curnum = $counter.Throughput.NextValue()
 	$maxnum = $counter.Bandwidth.NextValue()
 
 	New-Object PSObject -Property @{"Util"=$((( $curnum * 8 ) / $maxnum ) * 100);"Name"=$counter.Name}
 }
`

