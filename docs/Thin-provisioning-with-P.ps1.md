---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1987
Published Date: 
Archived Date: 2010-07-21t21
---

# thin provisioning with p - 

## Description

thin provisioning with powercil 4.0 version

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function ConvertVMDiskToThin($vm, $datastore) {
    $vmView = Get-View $vm
    $dsView = Get-View $datastore
    
    $relocateSpec = New-Object VMware.Vim.VirtualMachineRelocateSpec
 	$relocateSpec.Datastore = $dsView.MoRef
 	$relocateSpec.Transform = "sparse"
    
    $vmView.RelocateVM($relocateSpec, $null)
 }
`

