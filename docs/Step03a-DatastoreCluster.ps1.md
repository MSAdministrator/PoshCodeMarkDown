---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5666
Published Date: 2015-01-07t13
Archived Date: 2015-01-09t22
---

# step03a-datastorecluster - 

## Description

script to export datastore information per datastore cluster to csv

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $VC1 = "Old vCenter"
 $DSCluster = "Datastore Cluster"
 
 Connect-VIServer "$VC1"
 
 Get-DatastoreCluster -name $DSCluster | Get-Datastore  | Select-object Name | Export-Csv $DSCluster.csv
 
 Disconnect-VIServer -server "$VC1" -Force -Confirm:$false
`

