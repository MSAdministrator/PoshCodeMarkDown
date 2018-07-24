---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4117
Published Date: 
Archived Date: 2013-04-24t01
---

#  - 

## Description

find who created a vm

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Connect-VIServer <vCenter server>
 if (-not (Get-PSSnapin VMware.VimAutomation.Core -ErrorAction SilentlyContinue)) {
 	Add-PSSnapin VMware.VimAutomation.Core
 }
 if (-not (Get-PSSnapin Quest.ActiveRoles.ADManagement -ErrorAction SilentlyContinue)) {
 	Add-PSSnapin Quest.ActiveRoles.ADManagement
 }
 
 $VMs = Get-Cluster -Name <Cluster name> | Get-VM | Sort Name
 $VM = $VMs | Select -First 1
 If (-not $vm.CustomFields.ContainsKey("CreatedBy")) {
 	Write-Host "Creating CreatedBy Custom field for all VM's"
 	New-CustomAttribute -TargetType VirtualMachine -Name CreatedBy | Out-Null
 }
 If (-not $vm.CustomFields.ContainsKey("CreatedOn")) {
 	Write-Host "Creating CreatedOn Custom field for all VM's"
 	New-CustomAttribute -TargetType VirtualMachine -Name CreatedOn | Out-Null
 }
 Foreach ($VM in $VMs){
 	If ($vm.CustomFields["CreatedBy"] -eq $null -or $vm.CustomFields["CreatedBy"] -eq ""){
 		Write-Host "Finding creator for $vm"
 		$Event = $VM | Get-VIEvent -Types Info | Where { $_.Gettype().Name -eq "VmBeingDeployedEvent" -or $_.Gettype().Name -eq "VmCreatedEvent" -or $_.Gettype().Name -eq "VmRegisteredEvent" -or $_.Gettype().Name -eq "VmClonedEvent"}
 		If (($Event | Measure-Object).Count -eq 0){
 			$User = "Unknown"
 			$Created = "Unknown"
 		} Else {
 			If ($Event.Username -eq "" -or $Event.Username -eq $null) {
 				$User = "Unknown"
 			} Else {
 				$User = (Get-QADUser -Identity $Event.Username).DisplayName
 				if ($User -eq $null -or $User -eq ""){
 					$User = $Event.Username
 				}
 				$Created = $Event.CreatedTime
 			}
 		}
 		Write "Adding info to $($VM.Name)"
 		Write-Host -ForegroundColor Yellow "CreatedBy $User"
 		$VM | Set-Annotation -CustomAttribute "CreatedBy" -Value $User | Out-Null
 		Write-Host -ForegroundColor Yellow "CreatedOn $Created"
 		$VM | Set-Annotation -CustomAttribute "CreatedOn" -Value $Created | Out-Null
 	}
 }
`

