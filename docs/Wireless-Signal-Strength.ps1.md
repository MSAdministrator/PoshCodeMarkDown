---
Author: josh popp
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5118
Published Date: 2015-04-25t20
Archived Date: 2015-01-31t20
---

# wireless signal strength - 

## Description

there are a couple scripts that parse netsh commands.  i didn’t see this one already done, so i couldn’t steal it.  i suppose i could use some regex or something simple to cut the whitespace, so feel free to “fix her up”, but this got the job done (of putting the netsh output into an object).  this is a snip from a larger script i wrote as a looping, recording monitor.  i used this guy’s script for inspiration

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
     $wlanraw = netsh wlan show interface
 
     $objWLAN = "" | Select-Object Name,SSID,BSSID,Channel,ReceiveRate,TransmitRate,Signal
 
     ForEach ($Line in $wlanraw) {
         
     	if ([regex]::IsMatch($Line,"    Name")) {
     		$objWLAN.Name = $Line -Replace "    Name                   : ",""
     		}
                
 	if ([regex]::IsMatch($Line,"    SSID")) {
 		$objWLAN.SSID = $Line -Replace"    SSID                   : ",""
     	       	}
                
          if ([regex]::IsMatch($Line,"    BSSID")) {
     	 	$objWLAN.BSSID = $Line -Replace"    BSSID                  : ",""
 		}
                
 	if ([regex]::IsMatch($Line,"    Channel")) {
     	   	$objWLAN.Channel = $Line -replace "    Channel                : ",""
 		}
                
 	if ([regex]::IsMatch($Line,"    Receive rate")) {
     	   	$objWLAN.ReceiveRate = $Line -replace "    Receive rate \(Mbps\)    : ",""
 		}   
                
 	if ([regex]::IsMatch($Line,"    Transmit rate")) {
     	   	$objWLAN.TransmitRate = $Line -replace "    Transmit rate \(Mbps\)   : ",""
 		}  
                
 	if ([regex]::IsMatch($Line,"    Signal")) {
     	   	$objWLAN.Signal = $Line -replace "    Signal                 : ",""
 		}
         	}
`

