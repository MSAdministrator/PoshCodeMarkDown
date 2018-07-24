---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5667
Published Date: 2015-01-07t13
Archived Date: 2015-01-15t20
---

# step03b-move-datastore-t - 

## Description

script to create datastore clusters and add datastores from csv

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $VC1 = "New vCenter"
 $DSCluster = "Datastore Cluster"
 
 Connect-VIServer "$VC1"
 
 Set-DatastoreCluster -DatastoreCluster $DSCluster -SdrsAutomationLevel FullyAutomated
 
 Import-CSV $DSCluster.csv | ForEach-Object {
 
 	$Datastore = $_.Name
 
 move-datastore -Datastore $Datastore -Destination $DSCluster
 
 Disconnect-VIServer -server "$VC1" -Force -Confirm:$false
`

