---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3983
Published Date: 2013-02-21t14
Archived Date: 2013-02-27t05
---

# more psdrives - 

## Description

two functions for create and remove additional drives.

## Comments



## Usage



## TODO



## function

`mount-userdrives`

## Code

`#
 #
 function Mount-UserDrives {
   <#
     .Synopsis
         Create additional user drives.
     .Description
         To remove drives created with this function use Remove-UserDrives.
     .Link
         New-PSDrive
         Remove-UserDrives
   #>
   [Enum]::GetNames([Environment+SpecialFolder]) | % {
     New-PSDrive -n $_ -ps FileSystem -r $([Environment]::GetFolderPath($_)) -s Global | Out-Null
   }
 }
 
 function Remove-UserDrives {
   <#
     .Synopsis
         Remove additional user drives.
     .Description
         Dismount drives which mounted with Mount-UserDrives.
     .Link
         Remove-PSDrive
         Mount-UserDrives
   #>
   [Enum]::GetNames([Environment+SpecialFolder]) | % {
     Remove-PSDrive -n $_ -ps FileSystem -s Global
   }
 }
`

