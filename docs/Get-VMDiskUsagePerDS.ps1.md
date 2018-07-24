---
Author: justin grote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6034
Published Date: 2016-09-30t23
Archived Date: 2016-05-17t09
---

# get-vmdiskusageperds - 

## Description

fetches the per-datastore disk utilization of a virtual machine. is thin-provisioning aware and reports only the thin provisioned used space for usedgb.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $views = (get-vm | get-view)
 $views += (get-template | get-view)
 
 $VMDatastores = get-datastore
 
 $views | foreach {
 
     $vm = $PSItem
 
      foreach ($VMDSUsage in $vm.storage.PerDatastoreUsage) {
         $dsReport = [ordered]@{}
         $VMDatastore = $VMDatastores | where { $_.ID -match "datastore-$($VMDSUsage.datastore.value)"}
         $dsReport.VMName = $vm.name
         $dsReport.Datastore = $VMDatastore.Name
         $dsReport.UsedGB = [math]::round(($VMDSUsage.committed / 1GB),2)
         $dsReport.ProvisionedGB = [math]::round((($VMDSUsage.committed + $VMDSUsage.Uncommitted) / 1GB),2)
         $dsReport.DatastoreUsagePct = [math]::round((($dsReport.UsedGB / $VMDatastore.CapacityGB) * 100),2)
         new-object PSObject -property $dsReport
     }
 
 
 }
`

