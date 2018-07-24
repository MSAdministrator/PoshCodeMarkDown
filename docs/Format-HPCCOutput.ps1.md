---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5908
Published Date: 2015-06-24t23
Archived Date: 2015-07-02t03
---

# format-hpccoutput - 

## Description

this takes the output from hp configuration collector for an eva and converts it to useful csvs for reporting.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 $hpccfile = "C:\users\jgrote\desktop\SWMALEVA01.xml"
 
 $xmlRAW = [xml](get-content $hpccfile)
 
 $xmlEVA = $xmlRAW.scanmastercollection.collectiondata.eva
 
 $evaDiskGroup = $xmlEVA.diskgroup.object
 
 $evaDisks = $xmlEVA.disk.object
 
 $evaVirtualDisks = $xmleva.virtualdisk.object
 
 $evaDiskGroup | export-csv -notypeinformation "$($xmlEVA.devicename)-diskgroups.csv" 
 $evaDisks | export-csv -notypeinformation "$($xmlEVA.devicename)-disks.csv" 
 $evaVirtualDisks | export-csv -notypeinformation "$($xmlEVA.devicename)-virtualdisks.csv"
`

