---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.52
Encoding: ascii
License: cc0
PoshCode ID: 424
Published Date: 2009-06-11t14
Archived Date: 2016-03-04t23
---

# performancehistory - 

## Description

another modification to get-performancehistory to allow it to work in powershell 1.0 and still show averages.

## Comments



## Usage



## TODO



## script

`get-performancehistory`

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 param( [int]$count=1, [int[]]$id=@((Get-History -count 1| Select Id).Id) )
 
 function FormatTimeSpan($ts) {
    if($ts.Minutes) {
       if($ts.Hours) {
          if($ts.Days) {
             return "{0:##}d {1:00}:{2:00}:{3:00}.{4:00000}" -f $ts.Days, $ts.Hours, $ts.Minutes, $ts.Seconds, [int](100000 * ($ts.TotalSeconds - [Math]::Floor($ts.TotalSeconds)))
          }
          return "{0:##}:{1:00}:{2:00}.{3:00000}" -f $ts.Hours, $ts.Minutes, $ts.Seconds, [int](100000 * ($ts.TotalSeconds - [Math]::Floor($ts.TotalSeconds)))
       }
       return "{0:##}:{1:00}.{2:00000}" -f $ts.Minutes, $ts.Seconds, [int](100000 * ($ts.TotalSeconds - [Math]::Floor($ts.TotalSeconds)))
    }
 }
 
 if($id.Count -eq 1) { $id = ($id[0])..($id[0]-($count-1)) } 
 
 Get-History -id $id | 
 ForEach {
    $msr = $null
    $cmd = $_
    $avg = 7; $len = 8; $count = 1
    
       $msr = Measure-Object -in $cmd.CommandLine -Line -Word -Char
 
 $min, $max = ([regex]"^\s*(?:(?<min>\d+)\.\.(?<max>\d+)\s+\||for\s*\([^=]+=\s*(?<min>\d+)\s*;[^;]+\-lt\s*(?<max>\d+)\s*;[^;)]+\)\s*{)").match( $cmd.CommandLine ).Groups[1,2] | % { [int]$_.Value }
 $count = $max - $min
 if($count -le 0 ) { $count = 1 }
    
    "" | Select @{n="Id";        e={$cmd.Id}},
                @{n="Duration";  e={FormatTimeSpan ($cmd.EndExecutionTime - $cmd.StartExecutionTime)}},
                @{n="Average";   e={FormatTimeSpan ([TimeSpan]::FromTicks( (($cmd.EndExecutionTime - $cmd.StartExecutionTime).Ticks / $count) ))}},
                @{n="Commmand";  e={$cmd.CommandLine}}
 } | 
 Foreach { 
 $avg = [Math]::Max($avg,$_.Average.Length);
 $len = [Math]::Max($len,$_.Duration.Length);  
 $_ } | Sort Id |
 #}
`

