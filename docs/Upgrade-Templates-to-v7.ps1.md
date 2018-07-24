---
Author: afokkema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1214
Published Date: 2012-07-15t07
Archived Date: 2012-01-09t01
---

# upgrade templates to v7 - 

## Description

more info about this script can be found here

## Comments



## Usage



## TODO



## function

`convert-templatetovm`

## Code

`#
 #
 
 Function Convert-TemplateToVM($template){
 	
 	Write-Host "Converting" $vm -ForegroundColor Yellow		
 	Set-Template $template -ToVM -RunAsync | out-null
 }
 
 Function PowerOn-VM($vm){
 
 	Start-VM -VM $vm -Confirm:$false -RunAsync
 	
 		do {
 			$vmview = get-VM $vm | Get-View
 			$status = $vmview.Guest.ToolsStatus
 	
 				Write-Host $vm is starting! -ForegroundColor Yellow
 				sleep 5
 	
 		}until(($status -match "toolsOld") -or ($status -match "toolsOk"))
 		
 		if ($status -match "Ok"){
 			$Startup = "Ok"}
 		elseif($status -match "toolsOld"){
 			$Startup = "ToolsOld"}
 		else{
 			$Startup = "Not Ready"}
 		
 		return $Startup
 
 }
 
 Function Check-ToolsStatus($vm){
 
 	$vmview = get-VM $vm | Get-View
 	$status = $vmview.Guest.ToolsStatus
 
 		if ($status -match "toolsOld"){
 			$vmTools = "Old"}
 		elseif($status -match "toolsNotRunning"){
 			$vmTools = "Not Running"}
 		else{
 			$vmTools = "Ok"}
 		return $vmTools
 }
 
 Function Check-VMHardwareVersion($vm){
 	$vmView = get-VM $vm | Get-View
 	$vmVersion = $vmView.Config.Version
 	$v4 = "vmx-04"
 	$v7 = "vmx-07"
 	
 		if ($vmVersion -eq $v4){
 			$vmHardware = "Old"}
 		elseif($vmVersion -eq $v7){
 			$vmHardware = "Ok"}		
 		else{Write-Host "Error!!" -ForegroundColor Red
 			$vmHardware = "Error"}
 		
 		return $vmHardware
 }
 
 Function PowerOff-VM($vm){
 		
 		sleep 20
 		Shutdown-VMGuest -VM $vm -Confirm:$false
 					
 			do {
 				$vmview = Get-VM $vm | Get-View
 				$status = $vmview.Guest.ToolsStatus
 	
 					Write-Host $vm is stopping! -ForegroundColor Yellow
 					sleep 5
 	
 			}until($status -match "toolsNotRunning") 
 			if ($status -match "toolsNotRunning"){
 			$Shutdown = "Ok"}
 			else{
 			$Shutdown = "Not Ready"}
 			return $Shutdown	
 }
 
 Function ConvertTo-Template($vm){
 	
 	Write-Host "Converting" $vm -ForegroundColor Yellow
 	$vmview = Get-VM $vm | Get-View
 	$vmview.MarkAsTemplate() | Out-Null	
 
 }
 
 Function Upgrade-VMHardware($vm){
 
 	$vmview = Get-VM $vm | Get-View
 	$vmVersion = $vmView.Config.Version
 	$v4 = "vmx-04"
 	$v7 = "vmx-07"
 
 		if ($vmVersion -eq $v4){
 			Write-Host "Version 4 detected" -ForegroundColor Red
 			
 			Write-Host "Upgrading Hardware on" $vm -ForegroundColor Yellow
 			Get-View ($vmView.UpgradeVM_Task($v7)) | Out-Null
 	}	
 }
 
 $vCenter = Read-Host "Enter your vCenter servername"
 Connect-VIServer $vCenter
 
 $tmpfile = "$env:temp\tmpfile.csv"
 $templates = Get-Template -Name * | Export-Csv -NoTypeInformation $tmpfile
 $csv = Import-CSV $tmpfile
 
 	foreach($item in $csv){
 		$template = $item.Name
 		
 			Convert-TemplateToVM $template
 	}
 	
 	foreach($item in $csv){	
 		$vm = $item.Name
 		$vmHardware = Check-VMHardwareVersion $vm
 		
 				
 			if ($vmHardware -eq "Ok"){
 				Write-Host $vm "is up to date" -ForegroundColor Green
 				ConvertTo-Template $vm
 			}
 			else{Write-Host "Hardware is old" -ForegroundColor Red
 				if(PowerOn-VM $vm -eq "Ok"){
 					Write-Host "PowerOn Complete" -ForegroundColor Green
 					sleep 10
 					$vmToolsStatus = Check-ToolsStatus $vm
 					
 					if($vmToolsStatus -eq "Old"){
 							Write-Host "The VMware Tools are old" -ForegroundColor Red
 							Sleep 20
  							
 							Get-VMGuest $vm | Update-Tools
 							Sleep 120
 							Write-Host "VMware Tools are installed on:" $vm -ForegroundColor Cyan
 								
 								$vmToolsStatus = Check-ToolsStatus $vm
 							
 								if($vmToolsStatus -eq "Ok"){
 									
 									$PowerOffVM = PowerOff-VM $vm
 									if($PowerOffVM -eq "Ok"){
 										Write-Host $vm "is down" -ForegroundColor Yellow
 										
 										Upgrade-VMHardware $vm
 										ConvertTo-Template $vm
 										Write-Host $vm "is up to date" -ForegroundColor Green
 									}
 								}
 					
 					}
 					else{
 						$PowerOffVM = PowerOff-VM $vm
 						if($PowerOffVM -eq "Ok"){
 							Upgrade-VMHardware $vm
 							ConvertTo-Template $vm
 							Write-Host $vm "is up to date" -ForegroundColor Green
 						}	
 					}
 				}
 			}
 	}
 Remove-Item $tmpfile -Confirm:$false
 Disconnect-VIServer -Confirm:$false
`

