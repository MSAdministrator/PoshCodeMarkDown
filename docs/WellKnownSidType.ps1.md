---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4559
Published Date: 2013-10-26t15
Archived Date: 2013-11-01t01
---

# wellknownsidtype - 

## Description

see http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [Enum]::GetNames([Security.Principal.WellKnownSidType]) | % {
   $itm = [Security.Principal.WellKnownSidType]::$_
   try {
     $sid = New-Object Security.Principal.SecurityIdentifier($itm, $null)
     $sid.Translate([Security.Principal.NTAccount]).Value
   }
   catch {}
 }
`

