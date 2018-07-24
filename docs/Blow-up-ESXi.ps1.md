---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1554
Published Date: 
Archived Date: 2009-12-26t18
---

# blow up esxi - 

## Description

blow up your esxi host. for entertainment purposes only.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 foreach ($i in 10..1) {
 	Set-VMHostAdvancedConfiguration -name Annotations.WelcomeMessage -value "This host will self destruct in $i"
 }
 Start-Sleep 10
 Set-VMHostAdvancedConfiguration -name Annotations.WelcomeMessage -value ""
`

