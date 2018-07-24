---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3522
Published Date: 2014-07-17t07
Archived Date: 2014-02-09t10
---

# get-mdatastorelist - 

## Description

a version of the vmware get-datastore cmdlet that filters out datastore we don’t want to put vms on.  it filters by type of datastore and our naming conventions for the datastore naming indicating what kind of data is on the datastore.  line 9 will have to be updated for your own environment.

## Comments



## Usage



## TODO



## function

`get-mdatastorelist`

## Code

`#
 #
 Function Get-mDatastoreList {
 param($Cluster)
 
 $VMH = get-vmhost -Location $cluster | Where-Object { ($_.ConnectionState -eq "Connected") -and ($_.PowerState -eq "PoweredOn") } | Select-Object -Property Name | Sort-Object Name | Select -Last 1 -Property Name
 
 $DSTs = Get-Datastore -VMHost $VMH.Name | Where-Object { ($_.Type -eq "VMFS") -and ($_.Name -notmatch "local") -and ($_.Name -notmatch "iso") -and ($_.Name -notmatch "template") -and ($_.Name -notmatch "datastore") -and ($_.Name -notmatch "temp") -and ($_.Name -notmatch "decom") -and ($_.Name -notmatch "_BED-PR") -and ($_.Name -notmatch "VMAX") -and ($_.Name -notmatch "NEW")  -and ($_.Name -notmatch "MAY-PR-LEG") -and ($_.Name -notmatch "BED-QA-LEG") }
 
 Write-Output $DSTs
 }
 
`

