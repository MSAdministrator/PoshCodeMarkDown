---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5418
Published Date: 2017-09-11t15
Archived Date: 2017-04-10t11
---

# lspci - 

## Description

analog of unix lspci basic functionality.

## Comments



## Usage



## TODO



## function

`get-pcidevices`

## Code

`#
 #
 Set-Alias lspci Get-PCIDevices
 
 function Get-PCIDevices {
   <#
     .NOTES
         Author: greg zakharov
   #>
   
   gci HKLM:\SYSTEM\CurrentControlSet\Enum\PCI | % {
     gp (gci $_.PSPath).PSPath | select @{
       N='DeviceID'; E={
         $i = ([Regex]"\d+").Matches($_.LocationInformation)
         '{0:x2}:{1:x2}.{2}' -f [Int32]$i[0].Value, [Int32]$i[1].Value, $i[2].Value
       }
     }, Class, @{
       N='Device'; E={$_.DeviceDesc}
     }, @{
       N='DriverDate'; E={
         ($script:r = gp (Join-Path HKLM:\SYSTEM\CurrentControlSet\Control\Class $_.Driver)).DriverDate
       }
     }, @{
       N='DriverVersion'; E={$r.DriverVersion}
     }
   } | sort DeviceID | ft -a
 }
`

