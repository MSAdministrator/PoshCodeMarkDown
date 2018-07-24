---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1144
Published Date: 
Archived Date: 2009-06-07t13
---

# begin block - 

## Description

a begin block

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Begin { 
 	$VMHost_UUID = @{ 
         Name = "VMHost_UUID" 
         Expression = { $_.Summary.Hardware.Uuid } 
     }
 	$XenHost_UUID = @{
 		Name = "XenHost_UUID"
 		Expression = { $_.Uuid }
 	} 
 }
`

