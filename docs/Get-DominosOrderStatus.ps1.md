---
Author: xcud.com
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1355
Published Date: 2010-09-30t12
Archived Date: 2014-08-30t04
---

# get-dominosorderstatus - 

## Description

gets the status of your dominos pizza order. your phone number is the only input parameter. this is the most important psh module you’ll ever import.

## Comments



## Usage



## TODO



## script

`get-dominosorderstatus`

## Code

`#
 #
 #
 
 function Get-DominosOrderStatus($phone_number) {
 	$url = "http://trkweb.dominos.com/orderstorage/GetTrackerData?Phone=$phone_number"
 	[xml]$content = (new-object System.Net.WebClient).DownloadString($url);
 	$statii = select-xml -xml @($content) `
 			   -Namespace @{dominos="http://www.dominos.com/message/"} `
 			   -XPath descendant::dominos:OrderStatus
 	if($statii.Count -gt 0) { $statii | %{ $_.Node } }
 	else { "No orders" }
 }
 
 Export-ModuleMember Get-DominosOrderStatus
`

