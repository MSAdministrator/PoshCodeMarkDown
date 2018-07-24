---
Author: angel-keeper
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1896
Published Date: 2011-06-03t16
Archived Date: 2017-05-22t04
---

# deleted-objects - 

## Description

remove the folder or folders from computer on your network.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param (
 $Computer,
 [String[]]$ObjectsDeleted
 )
 
 $Info = $null
 $Disks = $null
 
 trap {Write-Host "Error WmiObject $Computer";Continue}
 $Disks += Get-WmiObject win32_logicaldisk -ComputerName $Computer | 
 		  Where-Object {$_.Size -ne $null}
 
 foreach ($Disk in $Disks){
 	
 	if ($Disk.Name -like "*:*") {
 	$Disk = $Disk.Name.Replace(":","$")
 	}
 	
 	trap {Write-Host "Error ChildItem $Computer";Continue}
 	$Info += Get-ChildItem "\\$Computer\$Disk\*" -Recurse -ErrorAction SilentlyContinue
 		
 	if ($Info){
 		
 		foreach ($Object in $ObjectsDeleted){
 			$Info | Where-Object {$_.Name -like $Object} | 
 					% {remove-item $_.fullname -Recurse -Force -ErrorAction SilentlyContinue}
 		}
 	}
 }
`

