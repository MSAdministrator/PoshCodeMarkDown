---
Author: angel-keeper
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1348
Published Date: 2010-09-28t20
Archived Date: 2016-07-30t12
---

# get-installedprogram - 

## Description

getting a list of installed programs on computers in the network except the following vendors

## Comments



## Usage



## TODO



## function

`get-installedprogram`

## Code

`#
 #
 function Get-InstalledProgram()
 {
 param (
 [String[]]$Computer,
 $User
 )
 #############################################################################################
 if ($User -is [String]) {
 	$Connection = Get-Credential -Credential $User
 }
 #############################################################################################
 foreach ($Comp in $Computer){
 	if ($Connection -eq $null){
 		$Install_soft = gwmi win32_product -ComputerName $Comp | 
 						where {$_.vendor -notlike "*Microsoft*" -and`
 						$_.vendor -notlike "*PGP*" -and $_.vendor -notlike "*Intel*" -and $_.vendor -notlike "*Corel*" -and $_.vendor -notlike "*Adobe*" -and $_.vendor -notlike "*ABBYY*" -and $_.vendor -notlike "*Sun*" -and $_.vendor -ne "SAP" -and $_.vendor -ne "Marvell" -and $_.vendor -ne "Hewlett-Packard"
 						} |
 						select __SERVER,Name,Version,InstallDate
 		$Install_soft
 	}
 	else {
 		$Install_soft = gwmi win32_product -ComputerName $Comp -Credential $Connection | 
 						where {$_.vendor -notlike "*Microsoft*" -and`
 						$_.vendor -notlike "*PGP*" -and $_.vendor -notlike "*Intel*" -and $_.vendor -notlike "*Corel*" -and $_.vendor -notlike "*Adobe*" -and $_.vendor -notlike "*ABBYY*" -and $_.vendor -notlike "*Sun*" -and $_.vendor -ne "SAP" -and $_.vendor -ne "Marvell" -and $_.vendor -ne "Hewlett-Packard"
 						} |
 						select __SERVER,Name,Version,InstallDate
 		$Install_soft
 	}
 }
 #############################################################################################
 }
`

