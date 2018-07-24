---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 100.0
Encoding: ascii
License: cc0
PoshCode ID: 2165
Published Date: 2011-09-09t21
Archived Date: 2016-05-29t12
---

# get-scriptperformancepro - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Computes the performance characteristics of a script, based on the transcript
 of it running at trace level 1.
 
 .DESCRIPTION
 
 To profile a script:
 
    1) Turn on script tracing in the window that will run the script:
       Set-PsDebug -trace 1
    2) Turn on the transcript for the window that will run the script:
       Start-Transcript
       (Note the filename that PowerShell provides as the logging destination.)
    3) Type in the script name, but don't actually start it.
    4) Open another PowerShell window, and navigate to the directory holding
       this script.  Type in '.\Get-ScriptPerformanceProfile <transcript>',
       replacing <transcript> with the path given in step 2.  Don't
       press <Enter> yet.
    5) Switch to the profiled script window, and start the script.
       Switch to the window containing this script, and press <Enter>
    6) Wait until your profiled script exits, or has run long enough to be
       representative of its work.  To be statistically accurate, your script
       should run for at least ten seconds.
    7) Switch to the window running this script, and press a key.
    8) Switch to the window holding your profiled script, and type:
       Stop-Transcript
    9) Delete the transcript.
 
 .NOTES
 
 You can profile regions of code (ie: functions) rather than just lines
 by placing the following call at the start of the region:
       Write-Debug "ENTER <region_name>"
 and the following call and the end of the region:
       Write-Debug "EXIT"
 This is implemented to account exclusively for the time spent in that
 region, and does not include time spent in regions contained within the
 region.  For example, if FunctionA calls FunctionB, and you've surrounded
 each by region markers, the statistics for FunctionA will not include the
 statistics for FunctionB.
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Path
 )
 
 Set-StrictMode -Version Latest
 
 function Main
 {
     $uniqueLines = @{}
     $samples = GetSamples $uniqueLines
 
     "Breakdown by line:"
     "----------------------------"
 
     $counts = @{}
     $totalSamples = 0;
     foreach($item in $samples.Keys)
     {
         $counts[$samples[$item]] = $item
         $totalSamples += $samples[$item]
     }
 
     foreach($count in ($counts.Keys | Sort-Object -Descending))
     {
         $line = $counts[$count]
         "{0,3}%: Line {1,4} -{2}" -f $percentage,$line,
             $uniqueLines[$line]
     }
 
     ""
     "Breakdown by marked regions:"
     "----------------------------"
     $functionMembers = GenerateFunctionMembers
 
     foreach($key in $functionMembers.Keys)
     {
         $totalTime = 0
         foreach($line in $functionMembers[$key])
         {
             $totalTime += ($samples[$line] * 100 / $totalSamples)
         }
 
         "{0,3}%: {1}" -f $percentage,$key
     }
 }
 
 function GetSamples($uniqueLines)
 {
     $logStream = [System.IO.File]::Open($Path, "Open", "Read", "ReadWrite")
     $logReader = New-Object System.IO.StreamReader $logStream
 
     $random = New-Object Random
     $samples = @{}
 
     $lastCounted = $null
 
     while(-not $host.UI.RawUI.KeyAvailable)
     {
         $sleepTime = [int] ($random.NextDouble() * 100.0)
         Start-Sleep -Milliseconds $sleepTime
 
         $rest = $logReader.ReadToEnd()
         $lastEntryIndex = $rest.LastIndexOf("DEBUG: ")
 
         if($lastEntryIndex -lt 0)
         {
             if($lastCounted) { $samples[$lastCounted] ++ }
             continue;
         }
 
         $lastEntryFinish = $rest.IndexOf("\n", $lastEntryIndex)
         if($lastEntryFinish -eq -1) { $lastEntryFinish = $rest.length }
 
         $scriptLine = $rest.Substring(
             $lastEntryIndex, ($lastEntryFinish - $lastEntryIndex)).Trim()
         if($scriptLine -match 'DEBUG:[ \t]*([0-9]*)\+(.*)')
         {
             $last = $matches[1]
 
             $lastCounted = $last
             $samples[$last] ++
 
             $uniqueLines[$last] = $matches[2]
         }
 
         $logReader.DiscardBufferedData()
     }
 
     $logStream.Close()
     $logReader.Close()
 
     $samples
 }
 
 function GenerateFunctionMembers
 {
     $callstack = New-Object System.Collections.Stack
     $currentFunction = "Unmarked"
     $callstack.Push($currentFunction)
 
     $functionMembers = @{}
 
     foreach($line in (Get-Content $Path))
     {
         if($line -match 'write-debug "ENTER (.*)"')
         {
             $currentFunction = $matches[1]
             $callstack.Push($currentFunction)
         }
         elseif($line -match 'write-debug "EXIT"')
         {
             [void] $callstack.Pop()
             $currentFunction = $callstack.Peek()
         }
         else
         {
             if($line -match 'DEBUG:[ \t]*([0-9]*)\+')
             {
                 if(-not $functionMembers[$currentFunction])
                 {
                     $functionMembers[$currentFunction] =
                         New-Object System.Collections.ArrayList
                 }
 
                 $hitLines = $functionMembers[$currentFunction]
                 if(-not $hitLines.Contains($matches[1]))
                 {
                     [void] $hitLines.Add($matches[1])
                 }
             }
         }
     }
 
     $functionMembers
 }
 
 . Main
`

