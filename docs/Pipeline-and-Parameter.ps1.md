---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1428
Published Date: 
Archived Date: 2009-12-17t03
---

# pipeline and parameter - 

## Description

sample by r_keith_hill

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param(
     [Parameter(ValueFromPipeline=$true, Mandatory=$true, Position=0)]
     [string[]]
     $ComputerName
 )
  
 Process {
     foreach ($cn in $ComputerName) {
         Write-Host "Processing $cn"
     }
 }
`

