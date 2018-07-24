---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2174
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t10
---

# invoke-advancedfunction. - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
     [Parameter(Mandatory = $true)]
     [ScriptBlock] $Scriptblock
     )
 
 & $scriptblock
`

