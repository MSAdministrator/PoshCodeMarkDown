---
Author: hal rottenberg
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 1549
Published Date: 2010-12-19t18
Archived Date: 2017-02-10t15
---

# new-linkedclone.ps1 - 

## Description

powercli script to create linked clones on an esx server (does require vcenter). this feature is not normally supported on esx, so this is a pretty nifty thing to do if you like living dangerously. info on linked clones

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 param (
 	[parameter(Mandatory=$true)][string]$SourceName,
 	[parameter(Mandatory=$true)][string]$CloneName
 )
 $vm = Get-VM $SourceName
 
 $cloneSnap = $vm | New-Snapshot -Name "Clone Snapshot"
 
 $vmView = $vm | Get-View
 
 $cloneFolder = $vmView.parent
 
 $cloneSpec = new-object Vmware.Vim.VirtualMachineCloneSpec
 $cloneSpec.Snapshot = $vmView.Snapshot.CurrentSnapshot
 
 $cloneSpec.Location = new-object Vmware.Vim.VirtualMachineRelocateSpec
 $cloneSpec.Location.DiskMoveType = [Vmware.Vim.VirtualMachineRelocateDiskMoveOptions]::createNewChildDiskBacking
 
 $vmView.CloneVM( $cloneFolder, $cloneName, $cloneSpec )
 
 Get-VM $cloneName
`

