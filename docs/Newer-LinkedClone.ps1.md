---
Author: cameron smith (original by hal rottenberg)
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 3016
Published Date: 2011-10-19t17
Archived Date: 2011-10-23t11
---

# newer-linkedclone.ps1 - 

## Description

adapted from new-linkedclone.ps1 from hal rottenberg.  this version takes a third parameter called snapname that allows a user to statically assign the snapshot that the clone should be built from.

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
 	[parameter(Mandatory=$true)][string]$CloneName,
 	[parameter(Mandatory=$true)][string]$SnapName
 )
 $vm = Get-VM $SourceName
 
 #$cloneSnap = $vm | New-Snapshot -Name "Clone Snapshot"
 
 $vmView = $vm | Get-View
 
 $cloneFolder = $vmView.parent
 
 $cloneSpec = new-object Vmware.Vim.VirtualMachineCloneSpec
 #$cloneSpec.Snapshot = $vmView.Snapshot.CurrentSnapshot
 $SnapRef = (Get-Snapshot -VM $SourceName -name "$SnapName") | Get-View
 $cloneSpec.Snapshot = $SnapRef.moref
 
 $cloneSpec.Location = new-object Vmware.Vim.VirtualMachineRelocateSpec
 $cloneSpec.Location.DiskMoveType = [Vmware.Vim.VirtualMachineRelocateDiskMoveOptions]::createNewChildDiskBacking
 
 $vmView.CloneVM( $cloneFolder, $cloneName, $cloneSpec )
 
 Get-VM $cloneName
`

