---
Author: skinny
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6238
Published Date: 2016-02-27t05
Archived Date: 2016-03-19t07
---

# scratchfolder creation - 

## Description

script creates unique scratch folders on the provided datastore for all esxi hosts in csv file and sets the advanced host settings to correspond to this location.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $Datastore = ""
 
 
 Import-CSV file.csv | ForEach-Object {
 $hostname = $_.vmhost
 
 connect-viserver $hostname -user root -password ""
 
 New-PSDrive -Name "mounteddatastore" -Root \ -PSProvider VimDatastore -Datastore (Get-Datastore $Datastore)
 
 Set-Location mounteddatastore:
 
 New-Item ".locker-$hostname" -ItemType directory
 
 CD G:
 Remove-PSDrive Mounteddatastore
 
 Set-VMHostAdvancedConfiguration -Name "ScratchConfig.ConfiguredScratchLocation" -Value "/vmfs/volumes/$Datastore/.locker-$hostname"
 
 Disconnect-VIServer -server $hostname -Force -Confirm:$false
 }
`

