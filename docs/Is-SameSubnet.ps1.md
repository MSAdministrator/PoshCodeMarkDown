---
Author: dragonmc77
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 527
Published Date: 2009-08-18t15
Archived Date: 2014-08-01t20
---

# is-samesubnet - 

## Description

simple function that determines whether a pair of ip’s are on the same subnet, given a specified mask.

## Comments



## Usage



## TODO



## function

`is-samesubnet`

## Code

`#
 #
 function Is-SameSubnet {param(	[string]$IP1,
 				[string]$IP2,
 				[string]$Subnet = "255.255.255.0")
 	
 	[string]$IP1Bits = ""
 	[string]$IP2Bits = ""
 	[string]$SubnetBits = ""
 									
 	[System.Net.IPAddress]::Parse($IP1).GetAddressBytes() |	
 		ForEach-Object {
 			$Bits = [Convert]::ToString($_, 2)
 			$IP1Bits += $Bits.Insert(0, "0"*(8-$Bits.Length))
 		}
 	[System.Net.IPAddress]::Parse($IP2).GetAddressBytes() | 
 		ForEach-Object {
 			$Bits = [Convert]::ToString($_, 2)
 			$IP2Bits += $Bits.Insert(0, "0"*(8-$Bits.Length))
 		}
 	[System.Net.IPAddress]::Parse($Subnet).GetAddressBytes() | 
 		ForEach-Object {
 			$Bits = [Convert]::ToString($_, 2)
 			$SubnetBits += $Bits.Insert(0, "0"*(8-$Bits.Length))
 		}
 	
 	if ($SubnetBits.StartsWith("11111111") -eq $FALSE) {return $FALSE}
 	
 	$BitPos = 0
 	do {
 		$Compare = [Convert]::ToInt32($IP1Bits.Substring($BitPos, 1)) -eq [Convert]::ToInt32($IP2Bits.Substring($BitPos, 1))
 		$Compare = [Convert]::ToString([Convert]::ToInt32($Compare))
 		[string]$SubnetBit = $SubnetBits.Substring($BitPos, 1)
 		
 		if ($SubnetBit -eq "1") {
 			if ($Compare -eq $SubnetBit) {$BitPos++} else {return $FALSE}
 		} else {break}
 	} until ($BitPos -eq $IP1Bits.Length)
 	return $TRUE
 }
`

