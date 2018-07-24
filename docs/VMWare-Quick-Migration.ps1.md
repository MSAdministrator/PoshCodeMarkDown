---
Author: justin grote
Publisher: 
Copyright: 
Email: 
Version: 0.2
Encoding: ascii
License: cc0
PoshCode ID: 3933
Published Date: 2013-02-03t10
Archived Date: 2013-02-12t14
---

# vmware quick migration - 

## Description

performs the functional equivalent of a hyper-v quick migration by suspending a vm, moving it to a new host, and resuming it. this does not require vmotion licensing. it works by providing required vm objects via the pipeline or the second argument, and specifying the destination host in the first argument, so you can use whatever query you want to build a list of vm’s to send to the command.

## Comments



## Usage



## TODO



## function

`quickmigrate-vm`

## Code

`#
 #
 #########################################
 #
 #
 
 #
 #########################################
 function QuickMigrate-VM {
 	PARAM(
 		$NewVMHost = ""
 		, [VMware.VimAutomation.Client20.VirtualMachineImpl]$VMsToMigrate = $null
 	)
 	BEGIN {
 		if (!$NewVMHost){
 			Write-Error "No Destination VMHost Specified. You must specify a host to migrate to. `n Example: Get-VM `"Test`" | QuickMigrate-VM `"Host1`""
 			break
 		}
 		elseif ($VMsToMigrate) {
 			Write-Output $InputObject | &($MyInvocation.InvocationName) -NewVMHost $newVMHost
 		}
 		else {
 			if ($NewVMHost.GetType().Name -eq "String") {
 				set-variable -name destinationVMHost -value (get-vmhost $NewVMHost) -scope 1
 			}
 			if (! $destinationVMHost -or (! $destinationVMHost.GetType().Name -eq "VMHostImpl")) {
 				write-error "Destination VM Host was not found or you provided the wrong object type. Please provide a VMHostImpl object or specify the fully qualified name of a VM Host"
 				break
 			}
 			write-host -fore white "===Begin Quick Migration==="
 		}
 	}
 	
 	PROCESS {
 		$step = 0
 		$skip = $false
 		trap {
 			write-host -fore yellow "`tSKIPPED: "$_.Exception.Message
 			set-variable -name skip -value ($true) -scope 1
 			continue
 		}
 		$vmToMigrate = $_
 		
 		if($_ -is [VMware.VimAutomation.Client20.VirtualMachineImpl]) {
 			write-host -fore white "Quick Migrating $($vmToMigrate.Name) to $NewVMHost..."
 		}
 		else {
 			throw "Object Passed was not a Virtual Machine object. Object must be of type VMware.VimAutomation.Client20.VirtualMachineImpl."
 		}
 		if (! $skip -and (get-cddrive $vmToMigrate).ConnectionState.Connected -ieq "TRUE") {
 			throw "Connected CD Drive. Please disconnect and try again."
 		}
 		if (! $skip -and (get-floppydrive $vmToMigrate).ConnectionState.Connected -ieq "TRUE") {
 			throw "Connected Floppy Drive. Please disconnect and try again."
 		}
 		
 		$sourceVMHost = get-vmhost -vm $vmToMigrate
 		if (! $skip -and ($sourceVMHost.Name -eq $destinationVMHost.Name)) {
 			throw "Source and Destination Hosts are the same."
 		}
 		
 		if (! $skip) {
 			$step++
 			write-host -fore cyan "`t$($step). Suspending $($vmToMigrate.Name)..."
 			$suspend = Suspend-VM -VM $vmToMigrate -confirm:$false
 		}
 		if (! $skip) {
 			$step++
 			write-host -fore cyan "`t($step). Moving $($vmToMigrate.Name) "to" $($destinationVMHost.Name)"
 			$migrate = Get-VM $vmToMigrate | Move-VM -Destination $destinationVMHost
 		}
 		if (! $skip) {
 			$step++
 			write-host -fore cyan "`t($step). Resuming" $vmToMigrate.Name"..."
 			$resume = Start-VM -VM $vmToMigrate
 		}
 		write-host -fore green "`tCOMPLETED"
 	}
 
 		END { write-host -fore white "===END Quick Migration===" }
 }
`

