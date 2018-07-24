---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3317
Published Date: 2013-04-06t06
Archived Date: 2013-09-16t06
---

# dumping com - 

## Description

this script dumping registred com objects names, sort and write them into a log.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $arr = @()
 $key = "HKLM:\SOFTWARE\Classes\CLSID"
 
 foreach ($i in (gci $key)) {
   $des = $key + "\" + $i.PSChildName + "\ProgID"
   Write-Progress "Dumping. Please, standby..." $des
 
   foreach ($a in (gp -ea 0 $des)."(default)") {
     $arr += $a
   }
 }
 
 [array]::Sort([array]$arr)
 $arr | Out-File -file C:\logs\COMnames.txt -enc UTF8
`

