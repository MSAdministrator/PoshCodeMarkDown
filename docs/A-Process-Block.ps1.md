---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1145
Published Date: 
Archived Date: 2009-06-07t13
---

# a process block - 

## Description

a process block

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Process { 
 	$InputTypeName = $_.GetType().Name 
     
 	if ( $InputTypeName -eq "VMHostImpl" ) { 
         	$output = $_ | Get-View | Select-Object $VMHost_UUID 
 	} elseif ($InputTypeName -eq "Host"){
 		$output = $_ | get-view | Select-Object $XenHost_UUID 
 	} else { 
 	        Write-Host "`nPlease pass this script either a VMHost or VM object on the pipeline.`n" 
 	} 
 }
`

