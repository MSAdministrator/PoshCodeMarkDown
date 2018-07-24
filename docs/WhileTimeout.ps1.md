---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 609
Published Date: 
Archived Date: 2008-09-29t01
---

# whiletimeout - 

## Description

generic wrapper for the while statement that will execute a condition a given number of max tries, waiting a given number of seconds in between.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function WhileTimeout ( [int]$interval, [int]$maxTries, [scriptblock]$condition )
 {
 	$i = 0
 	$startTime = Get-Date
 	while ( &$condition ) {
 		$i++
 		if ( $i -lt $maxTries ) {
 			Start-Sleep -seconds $interval
 		} else {
 			Throw "Operation exceeded timeout"
 		}
 	}
 	$endTime = Get-Date
 	$duration = ( $endTime - $startTime ).TotalSeconds
 	Write-Verbose "Operation elapsed time: $duration seconds"
 }
`

