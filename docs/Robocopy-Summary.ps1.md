---
Author: steele stenger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4686
Published Date: 2013-12-10t17
Archived Date: 2013-12-13t12
---

# robocopy summary - 

## Description

quickly pull files copied and size from robocopy log summary

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Function GetTOCStats {
     $TOCsummary = get-content -path "$env:USERPROFILE\desktop\testReport.log"
     [array]::Reverse($TOCsummary)
     [int] $i = "0"
     [string] $strTotal =  "Total"
     [string] $strCopied = "Copied"
     $TOCsummary | % {
         if (Select-String -Pattern "Total    Copied   Skipped  Mismatch    FAILED    Extras" -InputObject $_ -Quiet) {
             $start = $TOCsummary[$i].IndexOf($strTotal) + $strTotal.length
             $end = $TOCsummary[$i].IndexOf($strCopied) + $strCopied.length
             $copiedColumnwidth = ($end - $start)
             $TOCfileCount = $TOCsummary[($i-2)].substring($start,$copiedcolumnwidth).trim()
             $TOCsize = $TOCsummary[($i-3)].substring($start,$copiedcolumnwidth).trim()
             write-host "Copy size: $TOCsize"
             }
             $i++
     }
 }
 
 GetTOCStats
`

