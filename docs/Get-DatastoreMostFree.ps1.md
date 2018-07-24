---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3582
Published Date: 2014-08-17t08
Archived Date: 2014-02-09t09
---

# get-datastoremostfree - 

## Description

finds the datastore within a cluster with the most free space.

## Comments



## Usage



## TODO



## function

`get-datastoremostfree`

## Code

`#
 #
 function Get-DatastoreMostFree ($Cluster, [switch] $Second)
 {
 <#
 .SYNOPSIS
 Return the datastore in the cluster with the most unprovisioned disk space.  This takes thin provisioning into account.
 
 .DESCRIPTION
 Queries all the datastores in the cluster and returns the datastore with the most free disk space.  Used in conjunction with a mass cloning of virtual machines script. (Clone_Template_fromCSV.ps1)
 
 .NOTES
  1- You will need to modify the 2nd to last line's with criteria for your own environment.  I'm filtering out anything that's designed to hold guest VMs, like any datastores hosting ISO files and template VMs.
  2- Error and constraint checking need to be added.  If you try to put a VM a full LUN it will fail.
  
 .LINK
 
 .EXAMPLE
 Get-DatastoreMostFree.ps1 clustername
 #>
 
 Set-Variable -Name ScriptDir -Value "\\vmscripthost201\repo" -Scope Local
 . $ScriptDir\Get-mDataStoreList.ps1
 
 $LUNSizeFreeMB = 300*1024
 
 $DSTs = Get-mDataStoreList $Cluster
 
 if (!$Second) {
 	$DST =  $DSTs | Where-Object { $_.FreeSpaceMB -gt $LUNSizeFreeMB } | Select Name,@{n="Provisioned";e={($_.extensiondata.summary.capacity - $_.extensiondata.summary.freespace + $_.extensiondata.summary.Uncommitted)}} | Sort Provisioned -Descending | Select-Object -Last 1 -Property Name }
 else {
 	$DST =  $DSTs | Where-Object { $_.FreeSpaceMB -gt $LUNSizeFreeMB } | Select Name,@{n="Provisioned";e={($_.extensiondata.summary.capacity - $_.extensiondata.summary.freespace + $_.extensiondata.summary.Uncommitted)}} | Sort Provisioned -Descending | Select-Object -Last 2 | Select-Object -First 1 -Property Name }
 
 write-output $dst
 
 }
`

