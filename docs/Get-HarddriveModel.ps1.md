---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5285
Published Date: 2014-07-05t13
Archived Date: 2014-07-08t16
---

# get-harddrivemodel - 

## Description

retrieves harddrive model name without wmi

## Comments



## Usage



## TODO



## function

`get-harddrivemodel`

## Code

`#
 #
 function Get-HarddriveModel {
   <#
     .NOTES
         Author: greg zakharov
   #>
   (gp (
       Join-Path $key (
         'Enum\' + (gp (Join-Path ($key = 'HKLM:\SYSTEM\CurrentControlSet') 'Services\Disk\Enum')).('0')
       )
     )
   ).FriendlyName
 }
`

