---
Author: mjohnson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6493
Published Date: 2016-08-29t09
Archived Date: 2016-11-05t09
---

# remove special char - 

## Description

this will recursively remove non-alphanumeric\decimal (via regex) characters from all folder and filenames. the decimals are left in tact for file extensions.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 gci 'c:\test\' -Recurse | % { Rename-Item $_.FullName $($_.Name -replace
 	'[^\w\.]','') }
`

