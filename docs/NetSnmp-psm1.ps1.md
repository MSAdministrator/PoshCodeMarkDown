---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1238
Published Date: 2009-07-28t12
Archived Date: 2016-11-07t08
---

# netsnmp.psm1 - 

## Description

beginnings of a powershell v2 module that serves as a convenience wrapper for the net-snmp executables.

## Comments



## Usage



## TODO



## function

`get-snmpvalue`

## Code

`#
 #
 $NetSnmp = Join-Path $env:programfiles "Net-SNMP\bin"
 	if ( -not ( Test-Path "$NetSnmp\snmpwalk.exe" ) ) {
 		Throw "Net-SNMP binaries not found in $NetSnmp. Please install to this folder `
 			or edit the NetSnmp variable as appropriate."
 	}
 
 function Get-SnmpValue {
 	param (
 		[Parameter( Position = 0, Mandatory = $true )] $Agent,
 		[Parameter( Position = 1, Mandatory = $true )] $OID,
 		$Port = 161,
 		$Community = "public",
 		$Version = "2c"
 	)
 	&"$NetSnmp\snmpwalk.exe" "-v$Version" "-c$Community" "${Agent}:$Port" "$OID"
 }
`

