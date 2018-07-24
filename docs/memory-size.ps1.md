---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5384
Published Date: 2015-08-27t11
Archived Date: 2015-01-31t20
---

# memory size - 

## Description

very grateful to greg zakharov for this example. good luck you, greg, in search of work.

## Comments



## Usage



## TODO



## function

`get-ramlength`

## Code

`#
 #
 function Get-RamLength {
   <#
     .NOTES
         Author: greg zakharov
   #>
   
   $raw = (
     reg query "HKLM\HARDWARE\RESOURCEMAP\System Resources\Physical Memory"
   )[-1][-1..-8]
   for ($i = 1; $i -lt $raw.Length; $i++) {
     $ram += $raw[$i..($i - 1)]
     $i++
   }
   '{0}Gb' -f [Math]::Round([Convert]::ToUInt32(-join $ram, 16) / 1Gb)
 }
`

