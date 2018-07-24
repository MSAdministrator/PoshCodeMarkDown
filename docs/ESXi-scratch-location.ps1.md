---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6202
Published Date: 2016-02-05t04
Archived Date: 2016-05-12t10
---

# esxi scratch location - 

## Description

update custom scratch location esxi 5.1

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $vCenter = Read-Host "Enter ESXi host FQDN"
 
 Connect-VIServer $vCenter
 
 $esxcli=get-esxcli -vmhost $vCenter
 $esxcli.storage.vmfs.extent.list() | ft volumename,VMFSUUID -autosize
 
 $VolumeName = Read-Host "Enter Disk VolumeName"
 
 Get-AdvancedSetting -Entity (Get-VMhost -Name $vCenter) -Name "ScratchConfig.ConfiguredScratchLocation" | Set-AdvancedSetting -Value "/vmfs/volumes/$VolumeName" -Confirm:$False
 
 Disconnect-VIServer -Confirm:$false
`

