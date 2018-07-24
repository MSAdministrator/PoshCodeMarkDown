---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.53
Encoding: ascii
License: cc0
PoshCode ID: 683
Published Date: 
Archived Date: 2009-01-05t17
---

# get-performancehistory 2 - 

## Description

this much more complicated version of get-performancehistory shows the approximate length of the command or script, as well as how long it took to run.  great for those “my script is shorter/faster/cooler” than yours bragging sessions on irc ... or whatever. lets you compare several commands by just running each of them and then calling get-performancehistory -count 4

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
 function Get-PerformanceHistory {
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
                @{n="Lines";     e={$msr.Lines}},
                @{n="Words";     e={$msr.Words}},
                @{n="Chars";     e={$msr.Characters}},
                @{n="Type";      e={if($com.Count -gt 0){"Script"}else{"Command"}}},
                @{n="Commmand";  e={$cmd.CommandLine}}
 } | 
 Foreach { 
 $avg = [Math]::Max($avg,$_.Average.Length);
 $len = [Math]::Max($len,$_.Duration.Length);  
 $_ } | Sort Id |
 Format-Table @{l="Duration";e={"{0,$len}" -f $_.Duration}},@{l="Average";e={"{0,$avg}" -f $_.Average}},Lines,Words,Chars,Type,Commmand -auto
 }
`

