---
Author: st3v3o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6351
Published Date: 2016-05-19t13
Archived Date: 2016-05-24t07
---

# enable/configure snmp - 

## Description

enables snmp via add-windowsfeature (if not already enabled)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $pmanagers = "ADD YOUR MANAGER(s)"
 $commstring = "ADD YOUR COMM STRING"
 
 Import-Module ServerManager
 
 $check = Get-WindowsFeature | Where-Object {$_.Name -eq "SNMP-Services"}
 If ($check.Installed -ne "True") {
 	Add-WindowsFeature SNMP-Services | Out-Null
 }
 
 If ($check.Installed -eq "True"){
 	reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\SNMP\Parameters\PermittedManagers" /v 1 /t REG_SZ /d localhost /f | Out-Null
 	$i = 2
 	Foreach ($manager in $pmanagers){
 		reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\SNMP\Parameters\PermittedManagers" /v $i /t REG_SZ /d $manager /f | Out-Null
 		$i++
 		}
 	Foreach ( $string in $commstring){
 		reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\SNMP\Parameters\ValidCommunities" /v $string /t REG_DWORD /d 4 /f | Out-Null
 		}
 }
 Else {Write-Host "Error: SNMP Services Not Installed"}
`

