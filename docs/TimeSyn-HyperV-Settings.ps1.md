---
Author: vinith menon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3708
Published Date: 2012-10-22t21
Archived Date: 2012-10-26t14
---

# timesyn hyperv settings - 

## Description

check hyper-v settings for time synchronization status from within a vm hosted on a hyper-v host

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 	
 	Write-Host "Time Synchronization Status" -ForegroundColor Yellow
 	Write-Host "_______________________________" -ForegroundColor Yellow
 	Write-Host
 	
 	$VMhost = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Virtual Machine\Guest\Parameters" | select -ExpandProperty physicalhostname
 	$VirtualMachineName = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Virtual Machine\Guest\Parameters" | select -ExpandProperty VirtualMachineName
 	$session = New-PSSession -ComputerName $vmhost
 	Invoke-Command -Session $session {
 	param($vmname)
 	$MgmtSvc = gwmi -namespace root\virtualization MSVM_VirtualSystemManagementService
 	$VM = gwmi -namespace root\virtualization MSVM_ComputerSystem |?{$_.elementname -match $vmname}
 	$TimeSyncComponent = gwmi -query "associators of {$VM} where ResultClass = Msvm_TimeSyncComponent" -namespace root\virtualization                
 	$TimeSyncSetting  = gwmi -query "associators of {$TimeSyncComponent} where ResultClass = Msvm_TimeSyncComponentSettingData" -namespace root\virtualization                
 	if ($TimeSyncSetting.EnabledState -eq 3 ){
 	Write-Host "Time Synchronzation Disabled"  -ForegroundColor Green 
 	}
 	else{ Write-Host "Time Synchronzation Enabled" -ForegroundColor Red }
 	} -Args $VirtualMachineName
 	Get-PSSession | Remove-PSSession
`

