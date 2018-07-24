---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1162
Published Date: 
Archived Date: 2009-06-27t20
---

# remove-myoldcomputers - 

## Description

removes ad objects matching the filter (name=$name*) older than $maxdaysold

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##
 #
 #
 ##
 param (
   [String] $Name=((whoami).Split('\')[1]),
   [Int32] $MaxDaysOld=20
 )
 
 
 $root = [ADSI]''
 $searcher = new-object System.DirectoryServices.DirectorySearcher($root)
 $searcher.filter = "(Name=$Name*)"
 $computers = $searcher.findall()
 $computers | % {
   $Comp = [ADSI]$_.Path  
   $LastChange = [DateTime]::Now - [DateTime][String]$Comp.WhenChanged
   if ($LastChange.TotalDays -gt $MaxDaysOld) {
     Write-Host Deleting $Comp.Name [$LastChangeTimeSpan.TotalDays days old]
     $Comp.psbase.DeleteTree()
   }
 }
`

