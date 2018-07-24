---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1269
Published Date: 
Archived Date: 2009-08-25t00
---

# find sharable storage - 

## Description

this function finds storage within vmware that can be shared between esx hosts.

## Comments



## Usage



## TODO



## function

`get-shareabledatastore`

## Code

`#
 #
 function Get-ShareableDatastore {
 	$datastores = Get-Datastore
 
 	$hosts = Get-VMHost | Get-View -property ConfigManager
 	$storageSystems = @()
 	foreach ($h in $hosts) {
 		$storageSystems += Get-View $h.ConfigManager.StorageSystem -Property StorageDeviceInfo
 	}
 
 	foreach ($dso in $datastores) {
 		$ds = $dso | Get-View -Property Info
 
 		$dsInfo = $ds.Info
 		if ($dsInfo -is [VMware.Vim.NasDatastoreInfo]) {
 			Write-Output $dso
 			continue
 		}
 
 		$firstExtent = $dsInfo.Vmfs.Extent[0]
 
 		foreach ($hss in $storageSystems) {
 			$lun = $hss.StorageDeviceInfo.ScsiLun | Where { $_.CanonicalName -eq $firstExtent.DiskName }
 
 			if ($lun) {
 				$adapterTopology = $hss.StorageDeviceInfo.ScsiTopology.Adapter |
 					Where { $_.Target |
 						Where { $_.Lun |
 							Where { $_.ScsiLun -eq $lun.key }
 						}
 					} | Select -First 1
 
 				$adapter = $hss.StorageDeviceInfo.HostBusAdapter | Where { $_.Key -eq $adapterTopology.Adapter }
 
 				if ($adapter -is [VMware.Vim.HostFibreChannelHba] -or $adapter -is [VMware.Vim.HostInternetScsiHba]) {
 					Write-Output $dso
 				}
 
 				break
 			}
 		}
 	}
 }
`

