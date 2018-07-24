---
Author: alexander petrovskiy                                                                              #
Publisher: alexander petrovskiy, softwaretestingusingpowershell.wordpress.com                                #
Copyright: © 2011 alexander petrovskiy, softwaretestingusingpowershell.wordpress.com. all rights reserved.   #
Email: 
Version: 4.1
Encoding: utf-8
License: cc0
PoshCode ID: 2918
Published Date: 2011-08-18t13
Archived Date: 2011-09-16t12
---

# esximgmt module sample 3 - 

## Description

this is to demonstrate how to shutdown the machines you started while be reading the sample 2.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #######################################################################################################################
 #######################################################################################################################
 param([string]$Server,
 	  [string]$User,
 	  [string]$Password,
 	  [string]$DatastoreName,
 	  [string]$MachinePrefix,
 	  [int]$OperationTimeout
 	  )
 cls
 Set-StrictMode -Version Latest
 Import-Module ESXiMgmt -Force;
 
 Connect-ESXi -Server $Server -Port 443 `
 	-Protocol HTTPS -User $User -Password $Password `
 	-DatastoreName $DatastoreName;
 
 $VerbosePreference = [System.Management.Automation.ActionPreference]::Continue;
 $VMs = Get-VM *
 
 foreach($vm in $VMs)
 {
 	if ($vm.Name -like "$($MachinePrefix)*" -and `
 	{
 		Write-Verbose "$($vm.Name) is stopping";
 		Stop-ESXiVM -Server $Server `
 			-User $User -Password $Password `
 			-Id (Get-ESXiVMId $vm);
 		sleep -Seconds $OperationTimeout;
 	}
 }
 		
 Disconnect-ESXi
 
`

