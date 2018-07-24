---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5665
Published Date: 2015-01-07t12
Archived Date: 2015-01-15t20
---

# step02b-import-vm-to-fol - 

## Description

import vcenter folder structure incl vm relations

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 $NewVC = "New vCenter"
 
 Connect-VIServer "$NewVC"
 
 $vmlist = Import-Csv "migratedvms.csv"
 move-vm -vm $vmlist[0].name -Location (get-view -id $vmlist[0].folder -Server $newVC|get-viobjectbyviview) -Server $NewVC
 
 Disconnect-VIServer -server "$NewVC" -Force -Confirm:$false
`

