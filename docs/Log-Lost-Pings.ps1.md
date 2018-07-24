---
Author: jacob saaby nielsen
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 607
Published Date: 2009-09-26t09
Archived Date: 2016-05-29t07
---

# log lost pings - 

## Description

this script uses /n software’s netcmdlets, specifically the cmdlet “send-ping”, to continually ping a host, and log to a file whenever connectivity is lost.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $PingedHost = $Args[0]
 $PingTime = $Args[1]
 $LogPath = 'c:\'
 $KillSwitch = 1
 
 while ($KillSwitch -ne 0)
 {
 	Send-Ping $PingedHost
 	if ($PingedHost.Status -ne "OK")
 		{
 			'Lost connectivity at: ' + $(Get-Date -format "dd-MM-yyyy @ hh:mm:ss") | Out-File $($LogPath + $(Get-Date -format "dd-MM-yyyy") + '.log') -noClobber -append
 			$Error.Clear()
 			Start-Sleep $PingTime
 		}
 	else
 		{
 			Start-Sleep $PingTime
 		}
 }
`

