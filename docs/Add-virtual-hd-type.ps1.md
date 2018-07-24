---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 4.1
Encoding: ascii
License: cc0
PoshCode ID: 2087
Published Date: 
Archived Date: 2010-08-22t00
---

# add virtual hd type - 

## Description

add a type property to the virtualharddisk object to associate a disks type…. too big to past into twitter

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Foreach ($VM in Get-VM)
 {
     Foreach ($HD in Get-HardDisk -VM $VM)
     {
         Add-Member -InputObject $HD `
             -Name 'Type' `
             -MemberType 'noteproperty' `
             -passthru `
             -Value $( `
                 $VM.ExtensionData.Config.Hardware.device |
                     Where-Object {$_.key -eq $HD.ExtensionData.ControllerKey} |
                     ForEach-Object {
                         $_.gettype().name
                     }
             ) 
     }
 }
`

