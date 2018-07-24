---
Author: mazakane
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1833
Published Date: 2010-05-11t07
Archived Date: 2015-08-05t01
---

# productivity calculator - 

## Description

this short script converts time settings and displays a “productivity” report with a goal of 81%.

## Comments



## Usage



## TODO



## script

`get-productivity`

## Code

`#
 #
 <#
 Author: mazakane
 This short script converts Time settings and displays a "productivity" Report with a goal of 81%
 
 Example:
 get-productivity -Start 8:00 -End 17:00
 #>
 
 function get-Productivity{
   param(
      $start,
      $end
      )
 $a = ([datetime]::Parse("$start"))
 $b = ([datetime]::Parse("$end"))
 $lb = "0:30" #(Half an hour Break for lunch)
 $Wrk = ($b-$a)
 
 Write-Host "`nTotal working hours without Lunch break: $worktime`n" 
                               }
 else {$worktime = ($b - $a)
 Write-Host "`nTotal working hours including Lunch break: $worktime`n" 
       }
 $total = [int]$worktime.TotalMinutes 
 
  
 $prod = (($total*81)/100)/60 
  
 Write-warning "`nTo be 81% productive you need to work at least: $prod hours..."
 }
`

