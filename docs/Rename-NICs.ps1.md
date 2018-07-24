---
Author: powershelluser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1452
Published Date: 2012-11-03t17
Archived Date: 2012-06-08t04
---

# rename nics - 

## Description

rename network adapters to their mac addresses

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $Shell = New-Object -com shell.application
 $NetCons = $Shell.Namespace(0x31)
 $NetCons.Items() | 
 	where {$_.Name -like 'Local Area Connection*'} | 
 		foreach{$AdapName=$_.Name; get-WmiObject -class Win32_NetworkAdapter | 
 			where-Object {$_.NetConnectionID -eq $AdapName} | 
 				foreach {$MAC=$_.MacAddress}
 					$_.Name=$MAC.replace(':','.')
 				}
`

