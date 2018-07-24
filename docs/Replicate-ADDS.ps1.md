---
Author: chris
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2445
Published Date: 2012-01-06t13
Archived Date: 2012-11-25t05
---

# replicate-adds - 

## Description

forces replication of all dcs in the current logon domain.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $null = Start-Transcript "$pwd\$([System.IO.Path]::GetFileNameWithoutExtension($MyInvocation.MyCommand.Definition)).log"
 if ( (Get-PSSnapin -Name Quest.ActiveRoles.ADManagement -ErrorAction silentlycontinue) -eq $null ) {
 	if ( (Get-PSSnapin -Name Quest.ActiveRoles.ADManagement -Registered -ErrorAction SilentlyContinue) -eq $null) {
 		Write-Error "You must install Quest ActiveRoles AD Tools to use this script!"
 	} else {
 		Write-Host "Importing QAD Tools"
 		Add-PSSnapin -Name Quest.ActiveRoles.ADManagement -ErrorAction Stop
 	}
 }
 Write-Host "Beginning ADDS Replication"
 Write-Host "=========================="
 Get-QADComputer -ComputerRole 'DomainController' | % {
 	Write-Host "Replicating $($_.Name)"
 	$null = repadmin /kcc $_.Name
 	$null = repadmin /syncall /A /e $_.Name
 }
 Write-Host "=========================="
 Write-Host "Completed ADDS Replication"
 Stop-Transcript
`

