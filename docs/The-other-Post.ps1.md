---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 964
Published Date: 
Archived Date: 2009-03-27t18
---

# the other post - 

## Description

http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $insParm = '/s /v"/qn /norestart"' 
 $updList = get-cluster -name $YouClusterNameHere | get-vm |
 	where-object {$_.powerstate -eq "PoweredON"} |
 		foreach-object { get-view $_.ID } |
 			where { $_.guest.toolsstatus -match "toolsOld" } 
 foreach ($uVM in $updList) 
 {
 	$uVM.name 
 	$uVM.UpgradeTools_Task($insParm) 
 	Start-sleep -s 30 
 }
`

