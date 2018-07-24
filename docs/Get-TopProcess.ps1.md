---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1225
Published Date: 2009-07-23t07
Archived Date: 2012-12-31t14
---

# get-topprocess - 

## Description

returns the top processes by cpu usage

## Comments



## Usage



## TODO



## function

`get-diskactivity`

## Code

`#
 #
 
 param( 
     [string] $sortCriteria = "Processor", 
     [int] $Count = 5
     ) 
 
 function main 
 { 
     $cpuPerfCounters = @{} 
     $ioOtherOpsPerfCounters = @{} 
     $ioOtherBytesPerfCounters = @{} 
     $ioDataOpsPerfCounters = @{} 
     $ioDataBytesPerfCounters = @{} 
     $processes = $null 
     $lastPoll = get-date 
      
     $lastSnapshotCount = 0 
     $lastWindowHeight = 0 
      
     $processes = get-process | sort Id 
 
     foreach($process in $processes) 
     { 
         $cpuPercent = @(for($i=0;$i -lt 10;$i++)
         { 
             get-cpuPercent $process
         }) | measure-object -average 
         
         [int]$Percent = $cpuPercent.Average
         #$process | add-member NoteProperty Disk $activity  -force
         $process | add-member NoteProperty Processor $Percent -force
 
      } 
      
      $output = $processes | sort -desc $sortCriteria | select -first  $Count
      $output | format-table Id,ProcessName,MainWindowTitle,WorkingSet 
          
 } 
 
 function get-diskActivity ( 
     $process = $(throw "Please specify a process for which to get disk usage.") 
     ) 
 { 
     $processName = get-processName $process 
      
     if(-not $ioOtherOpsPerfCounters[$processName]) 
     { 
         $ioOtherOpsPerfCounters[$processName] = new-object System.Diagnostics.PerformanceCounter("Process","IO Other Operations/sec",$processName)
     } 
     if(-not $ioOtherBytesPerfCounters[$processName]) 
     { 
         $ioOtherBytesPerfCounters[$processName] = new-object System.Diagnostics.PerformanceCounter("Process","IO Other Bytes/sec",$processName) 
     } 
     if(-not $ioDataOpsPerfCounters[$processName]) 
     { 
         $ioDataOpsPerfCounters[$processName] = new-object System.Diagnostics.PerformanceCounter("Process","IO Data Operations/sec",$processName)
     } 
     if(-not $ioDataBytesPerfCounters[$processName]) 
     { 
         $ioDataBytesPerfCounters[$processName] = new-object System.Diagnostics.PerformanceCounter("Process","IO Data Bytes/sec",$processName)
     } 
 
 
     trap { continue; } 
 
     $ioOther = (100 * $ioOtherOpsPerfCounters[$processName].NextValue()) + ($ioOtherBytesPerfCounters[$processName].NextValue()) 
     $ioData = (100 * $ioDataOpsPerfCounters[$processName].NextValue()) + ($ioDataBytesPerfCounters[$processName].NextValue()) 
      
     return [int] ($ioOther + $ioData)     
 } 
 
 function get-cpuPercent ( 
     $process = $(throw "Please specify a process for which to get CPU usage.") 
     ) 
 { 
     $processName = get-processName $process 
      
     if(-not $cpuPerfCounters[$processName]) 
     { 
         $cpuPerfCounters[$processName] = new-object System.Diagnostics.PerformanceCounter("Process","% Processor Time",$processName)
     } 
 
     trap { continue; } 
 
     $cpuTime = ($cpuPerfCounters[$processName].NextValue() / $env:NUMBER_OF_PROCESSORS) 
     return [int] $cpuTime 
 } 
 
 function get-processName ( 
     $process = $(throw "Please specify a process for which to get the name.") 
     ) 
 { 
     $errorActionPreference = "SilentlyContinue" 
 
     $processName = $process.ProcessName 
     $localProcesses = get-process -ProcessName $processName | sort Id 
      
     if(@($localProcesses).Count -gt 1) 
     { 
         $processNumber = -1 
         for($counter = 0; $counter -lt $localProcesses.Count; $counter++) 
         { 
             if($localProcesses[$counter].Id -eq $process.Id) { break } 
         } 
          
         $processName += "#$counter" 
     } 
      
     return $processName 
 } 
 
 . main
`

