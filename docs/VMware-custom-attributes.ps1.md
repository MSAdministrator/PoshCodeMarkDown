---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4153
Published Date: 2016-05-13t12
Archived Date: 2016-03-04t23
---

# vmware custom attributes - 

## Description

gets creator, creation date and lastbackup information per vm and adds it as values of a custom attribute

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 if (-not (Get-PSSnapin VMware.VimAutomation.Core -ErrorAction SilentlyContinue)) {
 	Add-PSSnapin VMware.VimAutomation.Core
 }
 if (-not (Get-PSSnapin Quest.ActiveRoles.ADManagement -ErrorAction SilentlyContinue)) {
 	Add-PSSnapin Quest.ActiveRoles.ADManagement
 }
 $VC1 = "VI server"
 Connect-VIServer "$VC1"
 
 $VMs = Import-Csv Clusters.csv | 
 	Foreach {Get-Cluster -Name $_.Name | Get-VM | Sort Name}
 $VM = $VMs | Select -First 1
 
 
 If (-not $vm.CustomFields.ContainsKey("CreatedBy")) {
 	Write-Host "Creating CreatedBy Custom field for all VM's"
 	New-CustomAttribute -TargetType VirtualMachine -Name CreatedBy | Out-Null
 }
 If (-not $vm.CustomFields.ContainsKey("CreatedOn")) {
 	Write-Host "Creating CreatedOn Custom field for all VM's"
 	New-CustomAttribute -TargetType VirtualMachine -Name CreatedOn | Out-Null
 }
 If (-not $vm.CustomFields.ContainsKey("LastBackup")) {
 	Write-Host "Creating LastBackup Custom field for all VM's"
 	New-CustomAttribute -TargetType VirtualMachine -Name LastBackup | Out-Null
 }
 Foreach ($VM in $VMs){
 	If ($vm.CustomFields["CreatedBy"] -eq $null -or $vm.CustomFields["CreatedBy"] -eq ""){
 		Write-Host "Finding creator for $vm"
 		$Event = $VM | Get-VIEvent -Types Info | Where {$_.Gettype().Name -eq "VmBeingDeployedEvent" -or $_.Gettype().Name -eq "VmCreatedEvent" -or $_.Gettype().Name -eq "VmClonedEvent"}
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
 Foreach ($VM in $VMs){
 		Write-Host "Finding snapshot for $vm"
 		$Event = $VM | Get-VIEvent -Username "Netapp-service-account" -MaxSamples 3 | where {$_.GetType().Name -eq "TaskEvent" -and $_.Info.Name -eq "RemoveSnapshot_task"}
 				$Result = $Event.CreatedTime
 		Write "Adding info to $($VM.Name)"
 		Write-Host -ForegroundColor Yellow "Snapshot $User"
 		$VM | Set-Annotation -CustomAttribute "LastBackup" -Value $Result | Out-Null}
 		
 	
 Disconnect-VIServer -server "$VC1" -Force -Confirm:$false
`

