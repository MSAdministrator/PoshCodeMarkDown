---
Author: george mauer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2790
Published Date: 2011-07-13t21
Archived Date: 2011-07-16t11
---

# open-solution - 

## Description

open the sln file in the given directory hierarchy. present a list if there is more than one.

## Comments



## Usage



## TODO



## function

`open-solution`

## Code

`#
 #
 function Open-Solution([string]$path = ".") {
   $sln_files = Get-ChildItem $path -Include *.sln -Recurse
   if($sln_files -eq $null) {
     Write-Host "No Solution file found"
     return
   }
     do {
       Write-Host "Please select file (or Ctrl+C to quit)"
       for($i=0;$i -lt $sln_files.Count;$i=$i+1) { Write-Host "($i) - $($sln_files[$i])" }
       $idx = Read-Host
       $sln = $sln_files[$idx]
     }until($sln)
   }
   else { $sln = $sln_files }
   Invoke-Item $sln
 }
`

