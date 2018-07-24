---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1112
Published Date: 2009-05-18t14
Archived Date: 2013-07-28t13
---

# test connectivity - 

## Description

having problems with my dsl connection, a simple pinging log script

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 
 $ping = new-object System.Net.NetworkInformation.Ping
 $isbad = $true;
 do {
 try {
     $Reply = $ping.send('www.yahoo.com')
     if ($Reply.status �ne �Success�) { $txt = "$(get-date) problem"  ; write-Host $txt ; $txt | out-File -append c:\downloads\jetstreamlog.txt} 
     else { 
         if ($isbad) {$isbad = $false;$txt = "$(get-date) RECOVERED";write-Host $txt ; $txt | out-File -append c:\downloads\jetstreamlog.txt  }
         $txt = "$(get-date) good" ;write-Host $txt }
     }
 catch {
     $isbad = $true;
     $txt = "$(get-date) EXCEPTION"  ; write-Host $txt ; $txt | out-File -append c:\downloads\jetstreamlog.txt
 }
 sleep 4
 }
 while ($true)
`

