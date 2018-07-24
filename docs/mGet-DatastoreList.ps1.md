---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2448
Published Date: 2011-01-07t12
Archived Date: 2014-02-09t10
---

# mget-datastorelist - 

## Description

a version of the vmware get-datastore cmdlet that filters out datastore we don’t want to use for vms by type of datastore and our naming conventions for the datastore naming indicating what kind of data is on the datastore.  line 9 will have to be updated for your own environment.

## Comments



## Usage



## TODO



## function

`mget-datastorelist`

## Code

`#
 #
 Function mGet-DatastoreList {
 param($Cluster)
 
 $VMH = get-vmhost -Location $cluster | Where-Object { ($_.ConnectionState -eq "Connected") -and ($_.PowerState -eq "PoweredOn") } | Select-Object -Property Name | Sort-Object Name | Select -Last 1 -Property Name
 
 $DSTs = Get-Datastore -VMHost $VMH.Name | Where-Object { ($_.Type -eq "VMFS") -and ($_.Name -notmatch "local") -and ($_.Name -notmatch "iso") -and ($_.Name -notmatch "template") -and ($_.Name -notmatch "CLD") -and ($_.Name -notmatch "TRX") }
 
 Write-Output $DSTs
 }
`

