---
Author: frankspierings
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2115
Published Date: 2011-09-01t10
Archived Date: 2016-02-26t11
---

# modified wol impl. - 

## Description

this function can send wol packages. note that this a modified version of code already floating online. you need to specify the mac address as a string. optionally use -ports (0,1000) to specify the udp ports. (use -verbose to show which pacakges are being send).

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function SendUdpWol {
 	#- http://wiki.wireshark.org/WakeOnLAN
 	#- http://en.wikipedia.org/wiki/Wake-on-LAN
 	#
 	
 	param (
 		[parameter(Mandatory=$true)]
 		[ValidateLength(17,17)]
 		[ValidatePattern("[0-9|A-F]{2}:[0-9|A-F]{2}:[0-9|A-F]{2}:[0-9|A-F]{2}:[0-9|A-F]{2}:[0-9|A-F]{2}")]
 		[String]
 		$MacAddress,
 		
 		[parameter(Mandatory=$false)]
 		[int[]]
 		$Ports=@(0,7,9)
 	)
 	
 	[int]$MAGICPACKETLENGTH=102 #'Constant' defining total magic packet length.
 	
 		[Byte]("0x" + $_) 
 	}
 	
 	6..($magicPacket.Length - 1) |% {
 		$magicPacket[$_]=$byteMac[($_%6)]		
 	}
 	
 	[System.Net.Sockets.UdpClient] $UdpClient = new-Object System.Net.Sockets.UdpClient
 	foreach ($port in $Ports) {
 		Write-Verbose $("Sending magic packet to {0} port {1}" -f $MacAddress,$port)
 	}
 	$UdpClient.Close()
 }
`

