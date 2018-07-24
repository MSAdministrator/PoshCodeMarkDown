---
Author: david chung
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3222
Published Date: 2012-02-10t18
Archived Date: 2012-02-13t11
---

# set-vmbuildcsvdeploy - 

## Description

build vms using csv file created by set-vmbuildcsv.ps1 gui script.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #######################################################################
 #
 #
 #
 ########################################################################
 
 param( [string] $VISRV, $CSVBUILD, $user, $password)
 
 if ($user -ne $null -or $password -ne $null) {
 	Connect-VIServer $VISRV -User $user -Password $password
 }
 Else {
 	Connect-VIServer $VISRV
 }	
 
 $VMs = Import-Csv $CSVBUILD
 Foreach ($VM in $VMs) {
 	New-VM -vmhost $VM.Host -Name $VM.VM_Name -Template $VM.Template -Datastore $VM.Datastore -OSCustomizationSpec $VM.CustSpec -Location $VM.Folder
 	if ($VM.DISK1 -ne "") {
 		$disk1 = Get-VM $VM.VM_Name | Get-HardDisk | Where {$_.Name -eq "Hard disk 1"}
 		Set-HardDisk -harddisk $disk1 -CapacityKB $VM.DISK1 -Confirm:$false
 	}
 	if ($VM.DISK2 -ne ""){
 		If ((Get-VM $VM.VM_Name | Get-HardDisk | where {$_.name -eq "Hard disk 2"}) -eq $null) {
 			New-HardDisk -VM $VM.VM_Name -CapacityKB $VM.DISK2 -Confirm:$false -ThinProvisioned 
 		}
 		Else {
 			$disk2 = Get-VM $VM.VM_Name | Get-HardDisk | Where {$_.Name -eq "Hard disk 2"}
 			Set-HardDisk -harddisk $disk2 -CapacityKB $VM.DISK2 -Confirm:$false
 		}
 	}
 	If ($VM.CPU -ne "") {
 		Set-VM -vm $VM.VM_Name -Numcpu $VM.CPU -Confirm:$false
 	}
 	if ($VM.RAM -ne "") {
 		Set-VM -VM $VM.VM_Name -MemoryMB $VM.RAM -Confirm:$false
 	}
 	if ($VM.Network -ne "") {
 		$vmnet = Get-VM $VM.VM_Name | Get-NetworkAdapter | where {$_.Name -eq "Network Adapter 1"} 
 		$vmnet | Set-NetworkAdapter -NetworkName $VM.Network -StartConnected:$true -Confirm:$false
 	}	
 	If ($VM.ResourcePool -ne "") {
 		Move-VM -VM $VM.VM_Name -Destination $VM.ResourcePool -Confirm:$false
 	}
 	If ($VM.Power -eq "ON"){
 		Start-VM -VM $VM.VM_Name
 	}
 }	
 $CSVBUILDAudit =@()
 Write-Host "Running Audit for deployed VMs...."
 Foreach ($VMCSV in $VMs) {
 	$VM = Get-VM -Name $VMCSV.VM_name
 	$Details = "" | Select-Object VM_Name, OS, CPU, RAM, Disk1, Disk2, Network, Host, Datastore, Folder, Resource_Pool
 	If ((($VM.harddisks | where {$_.name -eq "hard disk 2"})) -eq $null) {
 		[string]$harddisk2 = $null
 	}
 	Else {
 	$harddisk2 = ($vm.harddisks | where {$_.name -eq "hard disk 2"}).capacitykb / 1mb	
 	}
 	$details.VM_name = $VM.name
 	$details.OS = $vm.extensiondata.summary.config.guestfullname
 	$details.CPU = $VM.numCPU
 	$details.RAM = $VM.memorymb
 	$details.Disk1 = ($vm.harddisks | where {$_.name -eq "hard disk 1"}).capacitykb / 1mb
 	$details.Disk2 = $harddisk2
 	$details.Network = ($vm.networkadapters | select networkname).networkname
 	$details.Host = ($vm.host | select name).name
 	$details.Datastore = ($vm.extensiondata.config.datastoreurl | select name).name
 	$details.Folder = ($vm.folder | select name).name
 	$details.Resource_Pool = ($vm.resourcepool | select name).name
 	$CSVBUILDAudit += $details
 }
 $File = "Audit" + $CSVBUILD
 $CSVBUILDAudit | Export-Csv $File -NoTypeInformation
 Write-Host "Completed.  See $File for VM deployment verfication."
 notepad $File
`

