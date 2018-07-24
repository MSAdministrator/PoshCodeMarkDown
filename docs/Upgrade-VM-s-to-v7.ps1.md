---
Author: afokkema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1217
Published Date: 2012-07-15t11
Archived Date: 2012-01-12t06
---

# upgrade vm's to v7 - 

## Description

more info can be found at www.ict-freak.nl

## Comments



## Usage



## TODO



## function

`poweron-vm`

## Code

`#
 #
 Function PowerOn-VM($vm){
 
 	Start-VM -VM $vm -Confirm:$false -RunAsync | Out-Null
 	
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
 
 Function PowerOff-VM($vm){
 		
 		sleep 20
 		Shutdown-VMGuest -VM $vm -Confirm:$false | Out-Null
 					
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
 
 Function CreateBeforeList{
 
 Write-Host "Creating a CSV File with VM info" -ForegroundColor Yellow
 
 $MyCol = @()
    ForEach ($vm in Get-Folder $Folder | Get-VM )
    {
    	$vmview = get-VM $vm | Get-View
    
       $myObj = "" | Select Host, VMName, PowerState, IPAddress, MacAddress, Nics, VMToolsVersion, VMToolsStatus, VMToolsRunningStatus
       $myObj.Host = $VM.Host.Name
       $myObj.VMName = $VM.Name
       $myObj.PowerState = $VM.PowerState
 	  $myObj.IPAddress = [String]($vmview.Guest.Net| ForEach {$_.IpAddress})
       $myObj.MacAddress = [String]($vmview.Guest.Net | ForEach {$_.MacAddress})
 	  $myObj.Nics = $vmview.Guest.Net.Count
 	  $myObj.VMToolsVersion = $vmview.Config.Tools.ToolsVersion
 	  $myObj.VMToolsStatus = $vmview.Guest.ToolsStatus
 	  $myObj.VMToolsRunningStatus = $vmview.Guest.ToolsRunningStatus
 	  
       $myCol += $myObj
     }
 	$myCol | Export-Csv C:\beforeHWchange.csv -NoTypeInformation 
 }
 
 Function CreateAfterList{
 
 Write-Host "Creating a CSV and an Excel File with VM info" -ForegroundColor Yellow
 
 $xlCSV = 6
 $xlXLS = 56
 $csvfile = "C:\afterHWchange.csv"
 $xlsfile = "C:\afterHWchange.xls"
 
 $Excel = New-Object -ComObject Excel.Application
 $Excel.visible = $True
 $Excel = $Excel.Workbooks.Add()
 
 $Sheet = $Excel.Worksheets.Item(1)
 $Sheet.Cells.Item(1,1) = "VMName"
 $Sheet.Cells.Item(1,2) = "IPAddress"
 $Sheet.Cells.Item(1,3) = "Settings"
 
 $intRow = 2
 
 $WorkBook = $Sheet.UsedRange
 $WorkBook.Interior.ColorIndex = 19
 $WorkBook.Font.ColorIndex = 11
 $WorkBook.Font.Bold = $True
 
 
 $import = Import-Csv "C:\beforeHWchange.csv"
 $vms = Get-Folder $Folder | Get-VM 
 
 foreach($vm in $vms){
 	$vmview = $vm | Get-View
 
 	foreach($item in $import){
 	$oldIp = $item.IPAddress
 	$newIP = $vm.Guest.IPAddress
 
 
 		if($item.VMName -eq $vm){
 			if($oldIp -eq $newIP){
 				
 				$Sheet.Cells.Item($intRow, 1) = [String]$vmview.Name 
 				$Sheet.Cells.Item($intRow, 2) = [String]$vm.Guest.IPAddress
 				$Sheet.Cells.Item($intRow, 3) = "Good"
 				$Sheet.Cells.Item($intRow, 3).Interior.ColorIndex = 4
 				}
 				else{
 				$Sheet.Cells.Item($intRow, 1) = [String]$vmview.Name 
 				$Sheet.Cells.Item($intRow, 2) = [String]$vm.Guest.IPAddress
 				$Sheet.Cells.Item($intRow, 3) = "Wrong"
 				$Sheet.Cells.Item($intRow, 3).Interior.ColorIndex = 3
 				
 				}
 		}
 	}
 
 
 $intRow = $intRow + 1}
 
 $WorkBook.EntireColumn.AutoFit()
 
 sleep 5
 
 $Sheet.SaveAs($xlsfile,$xlXLS)
 $Sheet.SaveAs($csvfile,$xlCSV) 
 
 }
 
 
 $vCenter = Read-Host "Enter your vCenter servername"
 $Folder = Read-Host "Enter the name of the folder where the VMs are stored"
 
 Connect-VIServer $vCenter
 
 $vms = Get-Folder $Folder | Get-VM  | Get-View | Where-Object {-not $_.config.template -and $_.Config.Version -eq "vmx-04" } | Select Name
 
 CreateBeforeList
 
 foreach($item in $vms){	
 		$vm = $item.Name
 		$vmHardware = Check-VMHardwareVersion $vm
 		$vmToolsStatus = Check-ToolsStatus $vm
 		
 			Write-Host "Hardware is old on:" $vm -ForegroundColor Red
 
 					if($vmToolsStatus -eq "Old"){
 							Write-Host "The VMware Tools are old on:" $vm -ForegroundColor Red
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
 										PowerOn-VM $vm
 										Write-Host $vm "is up to date" -ForegroundColor Green
 									}
 								}
 					
 					}
 					else{
 						$PowerOffVM = PowerOff-VM $vm
 						if($PowerOffVM -eq "Ok"){
 							Upgrade-VMHardware $vm
 							PowerOn-VM $vm
 							Write-Host $vm "is up to date" -ForegroundColor Green
 							
 						}	
 					}
 }
 
 Sleep 40
 
 CreateAfterList
 
 Disconnect-VIServer -Confirm:$false
`

