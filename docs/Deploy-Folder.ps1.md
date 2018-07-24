---
Author: hughs
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3018
Published Date: 2012-10-21t10
Archived Date: 2016-04-27t16
---

# deploy folder - 

## Description

this will pull a list of computers from ad and copy a folder to that system.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Clear-Host
 	{
 		$ObjComputerName = New-Object PSObject
 		$ObjComputerName = $Computer.name
 		$System = $ObjComputerName
 			{
 					{
 						Copy-Item -Path "C:\Users\Public\Training" -Destination "\\$System\C$\Users\Public" -Recurse -Force
 						Write-Host "Finished Win7 System $System"
 						$Finished += $System
 					}
 				ELSE 
 					{ 	
 						IF (Test-Path "\\$System\C$\Documents and Settings\All Users")
 							{
 								Copy-Item -Path "C:\Users\Public\Training" -Destination "\\$System\C$\Documents and Settings\All Users" -Recurse -Force
 								Write-Host "Finished WinXP System $System"
 								$Finished += $System
 							}
 					}
 			}
 		ELSE
 			{
 				$Skipped += $System
 			}
 	}
 	{
 		IF (Test-Connection -ComputerName $System -Quiet -Count 1)
 			{
 				IF (Test-Path "\\$System\C$\Users\Public")
 					{
 						Copy-Item -Path "C:\Users\Public\Training" -Destination "\\$System\C$\Users\Public" -Recurse -Force
 						Write-Host "Finished Win7 System $System"
 						$Finished += $System
 					}
 				ELSE 
 					{ 	
 						IF (Test-Path "\\$System\C$\Documents and Settings\All Users")
 							{
 								Copy-Item -Path "C:\Users\Public\Training" -Destination "\\$System\C$\Documents and Settings\All Users" -Recurse -Force
 								Write-Host "Finished WinXP System $System"
 								$Finished += $System
 							}
 					}
 			}
 		ELSE
 			{
 				$Offline +=$System
 			}
 	}
 Write-Host "Offline Systems:	$Offline"
 Write-Host "Finished Systems:	$Finished"
`

