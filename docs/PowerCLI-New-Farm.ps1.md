---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1547
Published Date: 
Archived Date: 2010-09-20t08
---

# powercli new-farm - 

## Description

this is a powercli sample script i wrote to demonstrate how you could entirely automate the creation of a relatively complex virtual farm environment.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Connect-VIServer vcenter.domain.com
 
 $rootfolder = Get-Folder -NoRecursion
 
 $dc = New-Datacenter -Name "vFarm" -Location $rootfolder
 
 $esxname = 1..10 | ForEach-Object { "atlesx{0:00}" -f $_ }
 
 $esxcred = Get-Credential
 
 $vmhost = $esxname | ForEach-Object { 
 	Add-VMHost -Name $_ -Credential $esxcred
 }
 
 $custname = "TGT", "WAL", "THD", "LOW", "KRO", "CVS", "TOY", "MAC", "SEA", "OLD"
 
 $rpool = for ( $i = 0; $i -lt $vmhost.length; $i++ ) {
 	New-ResourcePool -Location $vmhost[$i] -Name $custname[$i] 
 }
 
 $roleinfo = @(
 	@{ Name = "ProxyServer"; Num = 2 },
 	@{ Name = "WebServer"; Num = 4 },
 	@{ Name = "AppServer"; Num = 2 },
 	@{ Name = "DBServer"; Num = 2 }
 )
 
 foreach ( $custpool in $rpool ) {
 	foreach ( $role in $roleinfo ) {
 		$rolepool = New-ResourcePool -Name $role["Name"] -Location $custpool
 		
 		1..$role["Num"] | ForEach-Object {
 			$vmname = $custpool.Name + "-" + $role["Name"] + "-$_"
 			
 			New-VM -Name $vmname -ResourcePool $rolepool -Template $role["Name"] 
 		}
 	}
 }
`

