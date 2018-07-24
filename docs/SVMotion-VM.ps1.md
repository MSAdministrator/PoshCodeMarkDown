---
Author: hal rottenberg
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 575
Published Date: 
Archived Date: 2008-09-15t21
---

# svmotion-vm - 

## Description

please note that the move-vm’s -datastore parameter ought to work but there seems to be a bug in it, hence the need for this script. **

## Comments



## Usage

get-vm | svmotion-vm -destination (get-datastore foo)

## TODO



## function

`svmotion-vm`

## Code

`#
 #
 
 function SVMotion-VM {
 	param(
 		[VMware.VimAutomation.Client20.DatastoreImpl]
 		$destination
 	)
 	Begin {
 		$datastoreView = get-view $destination
 		$relocationSpec = new-object VMware.Vim.VirtualMachineRelocateSpec
 		$relocationSpec.Datastore = $datastoreView.MoRef
 	}
 	Process {
 		if ( $_ -isnot VMware.VimAutomation.Client20.VirtualMachineImpl ) {
 			Write-Error "Expected VMware object on pipeline. skipping $_"
 			continue
 		}
 		$vmView = Get-View $_
 		$vmView.RelocateVM_Task($relocationSpec)
 	}
 }
`

