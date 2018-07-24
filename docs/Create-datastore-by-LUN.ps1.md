---
Author: ed goad
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3412
Published Date: 2012-05-14t08
Archived Date: 2012-05-16t21
---

# create datastore by lun - 

## Description

creates a vmfs datastore via powershell by specifying the target lun id

## Comments



## Usage



## TODO



## function

`new-datastorebylun`

## Code

`#
 #
 function New-DatastoreByLun { param( [string]$vmHost, [string]$hbaId, [int]$targetId, [int]$lunId, [string]$dataStoreName )
 
   $view = Get-VMHost $vmHost | get-view
 
   $lun = $view.Config.StorageDevice.ScsiTopology | ForEach-Object { $_.Adapter } | Where-Object {$_.Key -match $hbaId} | ForEach-Object {$_.Target} | Where-Object {$_.Target -eq $targetId} | ForEach-Object {$_.Lun} | Where-Object {$_.Lun -eq $lunId}
 
   $scsiLun = Get-VMHost $vmHost | Get-ScsiLun | Where-Object {$_.Key -eq $lun.ScsiLun}
 
   New-Datastore -VMHost $vmHost -Name $dataStoreName -Path $scsiLun.CanonicalName -Vmfs -BlockSizeMB 8 -FileSystemVersion 3
 }
`

