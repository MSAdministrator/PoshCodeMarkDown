---
Author: lance robingeson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1147
Published Date: 
Archived Date: 2009-06-07t06
---

# get # of web connections - 

## Description

uses performancecounter to return the number of connections to web sites on the current machine.

## Comments



## Usage



## TODO



## function

`get-webserviceconnections`

## Code

`#
 #
 function Get-WebServiceConnections()
 {
   $results = @{}
   $perfmon = new-object System.Diagnostics.PerformanceCounter
   $perfmon.CategoryName = "Web Service"
   $perfmon.CounterName = "Current Connections"
 
   $cat = new-object System.Diagnostics.PerformanceCounterCategory("Web Service")
   $instances = $cat.GetInstanceNames()
 
   foreach ($instance in $instances)
   {
     $perfmon.InstanceName = $instance
     $results.Add($instance, $perfmon.NextValue())
   }
   write-output $results
 }
`

