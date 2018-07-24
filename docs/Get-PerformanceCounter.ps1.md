---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2475
Published Date: 2011-01-23t12
Archived Date: 2017-05-22t05
---

# get-performancecounter - 

## Description

get one or more performance counter objects.

## Comments



## Usage



## TODO



## function

`get-performancecounter`

## Code

`#
 #
 function Get-PerformanceCounter {
   <#
     .Synopsis
       Gets one or more Performance Counter objects.
     .Description
       This function will use .NET to retrieve a single Counter or all counters 
       for a particular Instance and Category.
     .Parameter category
       The Performance Counter Category.
     .Parameter instance
       (optional) If the Category is Multi-Instance, the Instance chosen.  For
       example, if you are querying the LogicalDisk category, the Instance would
       be the drive letter.  When querying a Multi-Instance Category, this
       option is mandatory.
     .Parameter counter
       (optional) The name of the Counter.  If you specify this option, only one
       counter will be returned.
     .Parameter computer
       (optional) The target computer.
     .Example
     Get-PerformanceCounter LogicalDisk C:
     .Example
     Get-PerformanceCounter LogicalDisk C: 'Current Disk Queue Length'
     .Notes
       NAME: Get-PerformanceCounter
       AUTHOR: Tim Johnson <tojo2000@tojo2000.com>
     .Links
     http://msdn.microsoft.com/en-us/library/system.diagnostics.performancecounter(v=VS.100).aspx
     http://msdn.microsoft.com/en-us/library/system.diagnostics.performancecountercategory.aspx
   #>
   param([Parameter(Mandatory = $true, Position = 0)]
         [string]$category,
         [Parameter(Mandatory = $false, Position = 1)]
         [string]$instance,
         [Parameter(Mandatory = $false, Position = 2)]
         [string]$counter,
         [Parameter(Mandatory = $false, Position = 3)]
         [string]$computer)
   
   try{
     if ($computer) {
       $perf = New-Object System.Diagnostics.PerformanceCounterCategory `
           $category $computer
     } else {
       $perf = New-Object System.Diagnostics.PerformanceCounterCategory `
           $category
     }
   } catch {
     Write-Host 'ERROR: Failed to create new Category object'
     Write-Host $Error[0].ToString()
     return
   }
   
   if (-not ($instance -or $counter)) {
     try {
       return $perf.GetCounters()
     } catch {
       Write-Host 'Unable to list counters for that Category.'
       Write-Host $Error[0].ToString()
       return
     }
   }
   
   if (-not $counter) {
     try {
       return $perf.GetCounters($instance)
     } catch {
       Write-Host 'Unable to list counters for that Category and Instance.'
       Write-Host $Error[0].ToString()
       return
     }
   }
   
   if (-not $computer) {
     $computer = '.'
   }
   
   if ($instance) {
     try {
       return New-Object System.Diagnostics.PerformanceCounter($category,
                                                               $counter,
                                                               $instance,
                                                               $computer)
     } catch {
       Write-Host 'Unable to retrieve that Counter.'
       Write-Host $Error[0].ToString()
       return
     }
   } else {
     try {
       return New-Object System.Diagnostics.PerformanceCounter($category,
                                                               $counter,
                                                               '',
                                                               $computer)
     } catch {
       Write-Host 'Unable to retrieve that Counter.'
       Write-Host $Error[0].ToString()
       return
     }
   }
 }
`

