---
Author: themoblin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5535
Published Date: 2014-10-24t10
Archived Date: 2014-12-27t21
---

# mailbox move monitor - 

## Description

run the following from the exchange 2010 shell and it will give you a semi-graphical view on the status of any open move requests

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 while ($TRUE) {
 	$Requests = get-moverequest|get-moverequeststatistics
 	$Requests | Foreach-Object {
 		if ($_.status -eq "InProgress") {
 			Write-Host -foregroundcolor "Yellow" "Mailbox move for $($_.Displayname) is in progress and is $($_.PercentComplete)% complete"
 		}
 		elseif ($_.status -eq "Completed"){
 			Write-Host -ForegroundColor "Green" "Mailbox move for $($_.Displayname) is complete. $($_.BaditemsEncountered) bad items were encountered"
 		}
 		elseif ($_.status -eq "Queued"){
 			Write-Host -ForegroundColor "RED" "Mailbox move for $($_.Displayname) is still queued"
 		}
 
 	}
 	Start-sleep -seconds 10
 	Clear-Host
 }
`

