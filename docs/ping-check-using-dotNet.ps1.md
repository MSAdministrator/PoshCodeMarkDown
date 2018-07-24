---
Author: jkavanagh58
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1420
Published Date: 
Archived Date: 2009-10-27t22
---

# ping check using dotnet - 

## Description

ping check using dotnet ping.  pieced together from existing examples on the web.  had to use $erroractionpreference = “silentlycontinue” to make it work on non-existing systems.  added to dns check to prevent a few more errors.  this is part of a script that queries an internal db and for every record does a ping to verify its online state

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function dnsref ($computername) {
 $ErrorActionPreference = "SilentlyContinue"
 $testrun=$Null
 trap { 
 	Write-Host "ERROR: $computername does not exist in DNS" -fore Yellow
 	Throw $_ }
 $testrun=[net.dns]::GetHostByName($computername) 
 if ($testrun -eq $Null){
 	Write-Host "No DNS Record" }
 else { 
 	foreach ($alias in $testrun){
  			PingX($alias.addresslist)
  		}
 	}
 }
 function PingX($ip) {
 	$ErrorActionPreference="SilentlyContinue"
 	$ping = New-Object System.Net.NetworkInformation.ping
 	if ($_Exception.Message -eq $null) {
 	$pingres = ($ping.send($ip)).Status | Out-Null
 	Write-Host $computername - $ip is REACHABLE -background "GREEN" -foreground "BLACk"}
 	else
 	{Write-Host $computername - $ip is NOT reachable  -background "RED" -foreground "BLACk"}
 	return $pingres
 }
`

