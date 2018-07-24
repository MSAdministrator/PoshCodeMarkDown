---
Author: adam bertram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4687
Published Date: 2013-12-11t03
Archived Date: 2013-12-13t12
---

# product code to guid - 

## Description

if you’re a developer, installation packager or a configmgr admin this script can be used to convert a product code to a guid.  this comes in handy when reverse engineering product installations.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $string_indexes = [ordered]@{0=8;8=4;12=4;16=2;18=2;20=2;22=2;24=2;26=2;28=2;30=2}
 $productcode = '1234567890123456789012345678901234'
 foreach ($index in $string_indexes.GetEnumerator()) {
     $part = $productcode.Substring($index.Key,$index.Value).ToCharArray()
     [array]::Reverse($part)
     $part -join ''
 }
`

