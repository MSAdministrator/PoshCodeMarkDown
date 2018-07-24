---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4790
Published Date: 2014-01-14t05
Archived Date: 2014-01-18t11
---

# format-hpevaxmlvdisk - 

## Description

this script takes the output from the “ls vdisk full xml > vdisks.xml” hp eva/p6000 and parses it into a form that can be imported into xml for reporting purposes.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $vdiskxmlfilepath = "vdisks.xml"
 
 $csvOutputFilepath = "vdisks.csv"
 
 $vdiskxml = [xml]("<vdisks>" + (get-content $vdiskxmlfile | where {$_ -notmatch "Active information"}) + "</vdisks>")
 
 $vdisks = $vdiskxml.vdisks.object
 
 $vdisksummary = $vdisks | select `
     requestedcapacity,
     allocatedcapacity,
     redundancy,
     comments
 
 $vdisksummary | Export-Csv $csvOutputFile -notypeinformation
`

