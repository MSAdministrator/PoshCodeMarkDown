---
Author: james pratt
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1317
Published Date: 2009-09-10t17
Archived Date: 2016-09-15t18
---

# networker - delete ssids - 

## Description

delete nw ssids by clientname , for use in adv_file environments.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 	Write-Host ""
 	Write-Host "This is dangerous - beware!"
 	Write-Host "Type: delssids client.domain.com to DELETE ALL it's SAVESETS!!"
 
 	function delssids {
 		$client = $args[0]
 		$ssids = (mminfo -av -q "client=$client" -r ssid)
 		$ssids | ForEach-Object { nsrmm -d -S $_ -y }
 		Write-Host "Removed SSID $_ "
 	}
`

