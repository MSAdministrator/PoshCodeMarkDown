---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2166
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-tomorrow.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 Set-StrictMode -Version Latest
 
 function GetDate
 {
     Get-Date
 }
 
 $tomorrow = (GetDate).AddDays(1)
 $tomorrow
`

