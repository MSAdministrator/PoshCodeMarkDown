---
Author: dragonmc77
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 599
Published Date: 2009-09-23t19
Archived Date: 2014-08-01t17
---

# ping-host - 

## Description

simple function that pings a host and returns a boolean.

## Comments



## Usage



## TODO



## function

`ping-host`

## Code

`#
 #
 function Ping-Host {param(	[string]$HostName,
 							[int32]$Requests = 3)
 	
 	for ($i = 1; $i -le $Requests; $i++) {
 		$Result = Get-WmiObject -Class Win32_PingStatus -ComputerName . -Filter "Address='$HostName'"
 		Start-Sleep -Seconds 1
 		if ($Result.StatusCode -ne 0) {return $FALSE}
 	}
 	return $TRUE
 }
`

