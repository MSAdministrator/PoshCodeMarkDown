---
Author: amber
Publisher: 
Copyright: 
Email: 
Version: 1.3.6
Encoding: ascii
License: cc0
PoshCode ID: 6622
Published Date: 2017-11-12t16
Archived Date: 2017-03-15t18
---

# send snmp trap - 

## Description

this will send an snmp trap to the specified manager (hostname). the script isn’t very flexible, but it’s a good example of how to build traps using sharpsnmplib. external requirementss

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param ( $DestinationHost )
 Add-Type -Path "C:\Program Files\LeXtudio Software\sharpsnmplib.dll"
 Add-Type -Path "C:\Program Files\LeXtudio Software\SharpSnmpLib.Controls.dll"
 $manager = [system.Net.Dns]::Resolve( $DestinationHost ).AddressList[0]
 $TrapDest = New-Object Net.IPEndPoint( $manager, 162 )
 
 [Lextm.SharpSnmpLib.Messaging.Messenger]::SendTrapV1( $TrapDest,
 	[IPAddress]::Loopback,
 	[Lextm.SharpSnmpLib.OctetString]"public",
 	[Lextm.SharpSnmpLib.ObjectIdentifier]"1.3.6",
 	[Lextm.SharpSnmpLib.GenericCode]::ColdStart,
 	0,
 	0,
 	( New-GenericObject.ps1 system.collections.generic.list lextm.sharpsnmplib.variable )
 )
`

