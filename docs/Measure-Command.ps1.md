---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5861
Published Date: 2015-05-15t22
Archived Date: 2015-05-26t00
---

# measure-command - 

## Description

measure-command, but with iterations and stuff

## Comments



## Usage



## TODO



## function

`measure-command`

## Code

`#
 #
 function Measure-Command {
     [CmdletBinding()]
     param(
         [ScriptBlock]$Command,
         [Int]$Iterations = 10,
         [Switch]$NoBuffer
     )
     
     if(!$NoBuffer) { $Null = & $Command }       
 
     $watch = [System.Diagnostics.Stopwatch]::new()
     $watch.Start()
     $Measurements = for($i=0; $i -lt $Iterations; $i++) {
             $1 = $watch.ElapsedTicks
             $Null = &$Command
             $2 = $watch.ElapsedTicks
             $2 - $1
     }
     $Measurements | 
         Measure-Object -Average -Minimum -Maximum | 
         &{
             process{
                 [PSCustomObject]@{
                     PSTypeName = "ExecutionPerformance"
                     Average = [Timespan]::FromTicks($_.Average)
                     Maximum = [Timespan]::FromTicks($_.Maximum)
                     Minimum = [Timespan]::FromTicks($_.Minimum)
                     Iterations = $Iterations
                     Command = "$Command"
                 }
             }
         }
 }
`

