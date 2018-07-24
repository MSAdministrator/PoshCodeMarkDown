---
Author: andy s
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 850
Published Date: 2009-02-04t13
Archived Date: 2016-06-03t06
---

# create-sccmcollection - 

## Description

this script will create a new system center configuration manager (sccm) 2007 collection.

## Comments



## Usage



## TODO



## function

`create-sccmcollection`

## Code

`#
 #
 
 function Create-SCCMCollection
 {
 param($Server = $Env:ComputerName, $Name, $Site, $ParentCollectionID = "COLLROOT")
 
 $ColClass = [WMIClass]"\\$Server\Root\SMS\Site_$($Site):SMS_Collection"
 $Col = $ColClass.PSBase.CreateInstance()
 $Col.Name = $Name
 $Col.OwnedByThisSite = $True
 $Col.Comment = "Collection $Name"
 $Col.psbase
 $Col.psbase.Put()
 
 $NewCollectionID = (Get-WmiObject -computerName $Server -namespace Root\SMS\Site_$Site -class SMS_Collection | where {$_.Name -eq $Name}).CollectionID
 				
 $RelClass = [WMIClass]"\\$Server\Root\SMS\Site_$($Site):SMS_CollectToSubCollect"
 $Rel = $RelClass.PSBase.CreateInstance()
 $Rel.ParentCollectionID = $ParentCollectionID
 $Rel.SubCollectionID = $NewCollectionID
 $Rel.psbase
 $Rel.psbase.Put()
 
 }
`

