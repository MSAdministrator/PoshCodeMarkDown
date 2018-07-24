---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3523
Published Date: 2015-07-17t07
Archived Date: 2015-03-08t13
---

# balance-datastores.ps1 - 

## Description

requires mget-datastorelist.ps1

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 -Change number of DSTs to is selection of $DSTMostFree to be variable.  Also add in logic to select DSTs with lower VMCounts.
 Total DSTs   Select DSTs
 ----------   -----------
    <5            1
    5-20          3
    >20           5
    
 -For $DSTLeastFree add in logic to select DSTs with higher VMCounts.
 #>
 
 param($Cluster,$Action)
 
 Write-Output "`n$(Get-Date)- Script started"
 
 If ( ($Action -ne "Move") -and ($Action -ne "Report") )  {
 	Write-Output "$(Get-Date)- Valid values for the parameter ""Action"" are either ""Move"" or ""Report"""
 	Write-Output "$(Get-Date)- Script aborted`n"
 	break
 }
 
 $IsClusterNameInvalid = $true
 Get-Cluster | % { If ($_.Name -eq $Cluster) {$IsClusterNameInvalid = $false} }
 If ($IsClusterNameInvalid) {
 	Write-Host "Error- Invalid Cluster Name" -Background Yellow -Foreground Red
 	Write-Host "Valid cluster names for this Virtual Center server."
 	Write-Host "---------------------------------------------------"
 	Get-Cluster | Sort
 	Write-Output "$(Get-Date)- Script aborted`n"
 	break
 }
 
 $ScriptDir = "\\vmscripthost201\repo"
 . $ScriptDir\Get-mDataStoreList.ps1
 If ($Cluster -match "Prod") { $DatastoreNumVMsLimit = 15 } Else { $DatastoreNumVMsLimit = 20 }
 $FreeSpacePercentMoveThreshold = 25
 
 $DSTs = Get-mDataStoreList $Cluster
 $DSTInfoAll = $DSTs | Select-Object Name,@{n="CapacityGB";e={[int](($_.CapacityMB/1024))}},@{n="FreeSpaceGB";e={[int](($_.FreeSpaceMB/1024))}},@{n="FreeSpacePercent";e={[int](($_.FreeSpaceMB/$_.CapacityMB*100))}},@{n="ProvisionedGB";e={[int](($_.ExtensionData.Summary.Capacity - $_.ExtensionData.Summary.Freespace + $_.ExtensionData.Summary.Uncommitted)/1024/1024/1024)}},@{n="ProvisionedPercent";e={[int](($_.ExtensionData.Summary.Capacity - $_.ExtensionData.Summary.Freespace + $_.ExtensionData.Summary.Uncommitted)/$_.ExtensionData.Summary.Capacity*100)}},@{n="VMCount";e={(Get-VM -Datastore $_ | Measure-Object).Count}}
 
 If     ($DSTInfoAll | Where-Object { $_.FreeSpacePercent -lt $FreeSpacePercentMoveThreshold } ) 
 	{ $DSTInfoLeastCandidates = $DSTInfoAll | Where-Object { $_.FreeSpacePercent -lt 25 } }
 ElseIf ($DSTInfoAll | Where-Object { $_.VMCount -gt $DatastoreNumVMsLimit } ) 
 	{ $DSTInfoLeastCandidates = $DSTInfoAll | Where-Object { $_.VMCount -gt $DatastoreNumVMsLimit } }
 Else   
 	{ $DSTInfoLeastCandidates = $DSTInfoAll }
 
 $DSTLeastFree = $DSTInfoLeastCandidates | Sort-Object FreespacePercent | Select-Object  -First 3 | Sort-Object ProvisionedPercent | Select-Object -Last 1
 $DSTMostFree  = $DSTInfoAll | Where-Object { $_.VMCount -lt $DatastoreNumVMsLimit } | Sort FreeSpacePercent | Select-Object -Last 3 | Sort-Object ProvisionedPercent | Select-Object -First 1
 
   #$DSTInfo | ft -a
   #$DSTLeastFree | ft -a
   #$DSTMostFree | ft -a
 
 $SourceVMsInitial = Get-VM -Datastore $DSTLeastFree.Name | Sort-Object UsedSpaceGB
 
 $SourceVMsNotExcludeded = $SourceVMsInitial | ForEach-Object { 
 	$vmtemp = $_.Name
 	$match = $false
 	Get-Content $ScriptDir\StaticInfo\sVMotion_ExcludeList.txt | ForEach-Object {
 		If ($vmtemp -eq $_) { $match = $true }
 	}
 	If ($match -eq $false) { $vmtemp }
 }
 
 $SourceVMs = $SourceVMsNotExcludeded | Where-Object { $_.MemoryMB -le 32768 } 
 
 $SourceVMCount = ($SourceVMs | Measure-Object).Count
 $SourceVMIndex = [int]($SourceVMCount/2)
 $SourceVMToMove = $SourceVMs[$SourceVMIndex]
 
 If ($Action -eq "Report" ) { Write-Output "+++++++ Reporting only +++++++" }
 
 $DSTLeastFree | Format-Table -AutoSize
 $DSTMostFree | Format-Table -AutoSize
 Get-VM $SourceVMToMove | Select Name,PowerState,VMHost,ResourcePool,NumCpu,MemoryMB,@{n="ProvisionedSpaceGB";e={[int]($_.ProvisionedSpaceGB)}},@{n="UsedSpaceGB";e={[int]($_.UsedSpaceGB)}} | Format-Table -AutoSize
 
 If ($Action -eq "Move" ) {
 	Write-Output "$(Get-Date)- *** Moving $($SourceVMToMove) from $(($DSTLeastFree).Name) to $(($DSTMostFree).Name)"
 	Move-VM -VM $SourceVMToMove -Datastore ($DSTMostFree).Name -Confirm:$false | Format-Table -AutoSize
 }
 
 Write-Output "$(Get-Date)- Script finished`n"
`

