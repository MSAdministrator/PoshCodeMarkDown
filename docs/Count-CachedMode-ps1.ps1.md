---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3410
Published Date: 
Archived Date: 2012-05-16t21
---

#  - 

## Description

count-cachedmode.ps1

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
 #
 
 
 $Script:CasLogUNCDir   ='\\<server>\<drv>\Web\Data\CASConnectLogs'
 
 $Clients = @{}
 
 foreach ($Line in (gci $Script:CasLogUNCDir | gc) ) {
     $LEDN = $Line.Split(",")[3]
     
     $OLVER = $Line.Split(",")[6]
     
     $Mode = $Line.Split(",")[7]
 
     if(-not ($Clients.ContainsKey($LEDN+'-'+$OLVer))) {
         $Clients.Add($($LEDN+'-'+$OLVer),$MODE)
     }
 }
 
 "Total Clients [ {0} ]; Cached [ {1} ]; Classic [ {2} ]"-f ($Clients.Count),(($Clients.getenumerator() | ?{$_.value-match'Cached'}).Count),(($Clients.getenumerator() | ?{$_.value-notmatch'Cached'}).Count)
`

