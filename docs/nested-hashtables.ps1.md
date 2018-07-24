---
Author: smaug9
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5763
Published Date: 2015-03-02t20
Archived Date: 2015-03-15t15
---

# nested hashtables - 

## Description

example using objects…

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Array1 = New-Object -TypeName PSObject -Property @{
             ArrayName = "Array01"
             Controller01Name = "vHBA1"
             Controller01WWN = "foobarfoobarfoobar01"
             Controller02Name = "vHBA2"
             Controller02WWN = "foobarfoobarfoobar02"        
         }
 
 $Array2 = New-Object -TypeName PSObject -Property @{
             ArrayName = "Array02"
             Controller01Name = "vHBA1"
             Controller01WWN = "foobarfoobarfoobar01"
             Controller02Name = "vHBA2"
             Controller02WWN = "foobarfoobarfoobar02"
         }
 
 $FabricA = @{
     SwitchIP = "1.2.3.4"
     VsanID = "Vsan01"
     ZoneSetname = "FabricA"
     Arrays = $Array1, $Array2
 }
 
 $FabricA.Arrays | Where {$_.arrayname -like '*01'}
 
`

