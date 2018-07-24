---
Author: bobsys
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4571
Published Date: 2013-10-30t22
Archived Date: 2013-11-04t00
---

# sdm gpae - 

## Description

adding group policy preferences printer with sdm gpae (group policy automation engine) including item-level targeting

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 
 Import-Module SDM-GroupPolicy
 
 
 $PrinterList=Import-CSV C:\scripts\printers.csv
 
 foreach ($entry in $PrinterList)
 {
 
 
 $Domain=$entry.Domain
 $GPOName=$entry.GPOName
 $PrinterName=$entry.PrinterName
 $PrinterPath=$entry.PrinterPath
 $Order=$entry.Order
 $GroupName=$entry.GroupName
 
 
 $gpo = Get-SDMgpobject -gpoName "gpo://$Domain/$GPONAME" -openByName
 
 
 $container = $gpo.GetObject("User Configuration/Preferences/Control Panel Settings/Printers/Shared Printer")
 
 
 $printer = $container.Settings.AddNew("$PrinterName")
 
 $printer.Put("Action",[GPOSDK.EAction]"Create")
 
 $printer.Put("Share path","$PrinterPath")
 
 
 $printer.Put("Order",$Order)
 
 
 
 $iilt = $gpo.CreateILTargetingList()
 
 $itm = $iilt.CreateIILTargeting([GPOSDK.Providers.ILTargetingType]"FilterGroup")
 
 
 $itm.Put("Group","$GroupName")
 
 $itm.Put("UserInGroup", $true)
 
 $iilt.Add($itm)
 
 
 $printer.put("Item-level targeting", $iilt)
 
 
 $printer.Save()
 }
`

