---
Author: andrew
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5960
Published Date: 2016-08-01t20
Archived Date: 2016-05-17t13
---

# dn to canonical - 

## Description

not all objects which looks like organizationalunit are them – some are containters (e.g. user, computers) – function corrected and extended to handle this kind of situations

## Comments



## Usage



## TODO



## function

`convertfrom-dn`

## Code

`#
 #
 function ConvertFrom-DN {
     
     
     
     param ([string]$DN = (Throw '$DN is required!'))
     
     foreach ($item in ($DN.replace('\,', '~').split(","))) {
         
         switch -regex ($item.TrimStart().Substring(0, 3)) {
             
             "CN=" { $CN +=, $item.replace("CN=", ""); $CN += '/'; continue }
             
             "OU=" { $OU +=, $item.replace("OU=", ""); $OU += '/'; continue }
             
             "DC=" { $DC += $item.replace("DC=", ""); $DC += '.'; continue }
             
         }
         
     }
     
     $canonical = $dc.Substring(0, $dc.length - 1)
     
     If ($ou.count -gt 0) {
         
 	for ($i = $ou.count; $i -ge 0; $i--) { $canonical += $ou[$i] }
         
     }
     
     If ($CN.count -gt 0) {
         
         for ($i = $CN.count; $i -ge 0; $i--) { $canonical += $CN[$i] }
         
     }
     
     return $canonical
 }
`

