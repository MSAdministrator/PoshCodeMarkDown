---
Author: robert bordeaux
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6270
Published Date: 2016-03-28t18
Archived Date: 2016-10-24t17
---

# dell open manage racadm - 

## Description

configure dell om racadm ip and msi install if needed.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ######################################
 #
 #
 #
 ######################################
 
 $node = 252
 $subnet = "255.255.255.0"
 $mask = $subnet.ToString()
 #$installer = "C:\Temp\OpenManage\windows\SystemsManagement\SysMgmt.msi"
 $racadmexe = "C:\Program Files\Dell\SysMgt\idrac\racadm.exe"
 
 $ServerOctetOne = ipconfig | where-object {$_ �match �IPv4 Address�} | foreach-object{$_.Split(�:�)[1].Trim()} | foreach-object{$_.Split(�.")[-4]} 
 $ServerOctetTwo = ipconfig | where-object {$_ �match �IPv4 Address�} | foreach-object{$_.Split(�:�)[1].Trim()} | foreach-object{$_.Split(�.")[-3]} 
 $ServerOctetThree = ipconfig | where-object {$_ �match �IPv4 Address�} | foreach-object{$_.Split(�:�)[1].Trim()} | foreach-object{$_.Split(�.")[-2]} 
 $ServerCFGNode = $ServerOctetOne + "." + $ServerOctetTwo + "." + $ServerOctetThree + "." + $node
 $cfgnode = $ServerCFGNode.ToString()
 #$ServerCFGNode.ToString() | Write-Host
 
 $Gateway = ipconfig | where-object {$_ �match �Default Gateway�} | foreach-object{$_.Split(�:�)[1].Trim()}
 $ServerGateway = $Gateway.ToString()
 #$ServerGateway | Write-Host
 
 
 
 
 
 & "C:\Program Files\Dell\SysMgt\idrac\racadm.exe" setniccfg -s $CFGNode $mask $ServerGateway
`

