---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4567
Published Date: 2013-10-28t07
Archived Date: 2013-11-01t01
---

# accelerators for posh 2 - 

## Description

as you know powershell v3 has [accelerators] type but powershell v2 has not this feature. so why do i have to endure this omission in powershell v2?

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $ta = [Type]::GetType("System.Management.Automation.TypeAccelerators")
 $ta::Get.Keys.GetEnumerator() | % {$arr = @()}{
   $arr += $($_ -ne 'accelerators')
 }{
   if (-not ($arr -contains 'False')) {
     $ta::Add('accelerators', $ta)
   }
 }
`

