---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 991
Published Date: 
Archived Date: 2017-05-24t00
---

#  - 

## Description

copy data between folders including a progressbar

## Comments



## Usage



## TODO



## script

`copy-data`

## Code

`#
 #
 ################################################################################################################
 
 #
 #
 #
 #
 
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
 
 function copy-data {
 	param(
 		[string]$file = "",
 		[string]$dest = "",
 		[switch]$Recurse,
 		[switch]$Confirm
 	)
 		
 $copy = cp $file $dest
 for ($a=1; $a -lt 100; $a++) {
 Write-Progress -Activity �Copying...� -SecondsRemaining $a -Status "% Complete:" -percentComplete $a
 }
 }
 
 
 set-Alias cpd copy-data
`

