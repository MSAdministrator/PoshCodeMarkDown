---
Author: cody bunch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1152
Published Date: 2009-06-10t11
Archived Date: 2013-10-16t19
---

# get-uuid_allhvs.ps1 - 

## Description

#the powershell talk

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 
 Begin { 
 	$VMHost_UUID = @{ 
         Name = "VMHost_UUID" 
         Expression = { $_.Summary.Hardware.Uuid } 
     }
 	$XenHost_UUID = @{
 		Name = "XenHost_UUID"
 		Expression = { $_.Uuid }
 	} 
 }
 
 Process { 
 	$InputTypeName = $_.GetType().Name 
     
 	if ( $InputTypeName -eq "VMHostImpl" ) { 
         $output = $_ | Get-View | Select-Object $VMHost_UUID 
 	} elseif ($InputTypeName -eq "Host"){
 		$output = $_ | Select-Object $XenHost_UUID 
 	} else { 
         Write-Host "`nPlease pass this script either a VMHost or VM object on the pipeline.`n" 
     }
 	$output
 }
`

