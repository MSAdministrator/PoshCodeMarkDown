---
Author: cody bunch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1169
Published Date: 2009-06-23t07
Archived Date: 2014-10-10t07
---

# reconfigure-ha.ps1 - 

## Description

reconfigure-ha.ps1	 – take a vmhost object from the pipeline and apply the ‘reconfigure ha host’ task

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Process {
     if ( $_ -isnot [VMware.VimAutomation.Client20.VMHostImpl] ) {
         Write-Error "VMHost expected, skipping object in pipeline."
         continue
     }
 	$vmhostView = $_ | Get-View
     $vmhostView.ReconfigureHostForDAS_Task()
 }
`

