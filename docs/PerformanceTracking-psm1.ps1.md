---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 2.5
Encoding: ascii
License: cc0
PoshCode ID: 1562
Published Date: 
Archived Date: 2010-01-06t03
---

# performancetracking.psm1 - 

## Description

a module for tracking performance of commands as you work. exports get-performancehistory that shows you the time and memory footprint of commands you’ve recently executed. you can query by id, or by count to show the most recent x commands.

## Comments



## Usage



## TODO



## script

`get-performancehistory`

## Code

`#
 #
 $Parser = [System.Management.Automation.PsParser]
 $script:lastMemory = Get-Process -id $PID
 $global:LastTweets = new-object System.Collections.Generic.List[PSObject]
 $global:PerformanceHistory = @{}
 
 if(!(Test-Path Variable:PrePerformanceTrackingPrompt -EA 0)){
    $Global:PrePerformanceTrackingPrompt = ${Function:Prompt}
 }
 
 
 function Global:Prompt {
    $currMemory = Get-Process -id $PID
    $lastCommand = Get-History -Count 1
    $global:PerformanceHistory[$lastCommand.Id] = Get-Performance $lastCommand @{
                                                    PrivateMemoryDiff = $currMemory.PrivateMemorySize64 - $lastMemory.PrivateMemorySize64
                                                    VirtualMemoryDiff = $currMemory.VirtualMemorySize64 - $lastMemory.VirtualMemorySize64
                                                    PrivateMemorySize64 = $currMemory.PrivateMemorySize64
                                                    VirtualMemorySize64 = $currMemory.VirtualMemorySize64
                                                  }
    if($PerformanceHistory.Count -gt $MaximumHistoryCount) {
       $null = $PerformanceHistory.Remove( ($lastCommand.Id - $MaximumHistoryCount) )
    }
    $lastMemory = $currMemory
    return &$PrePerformanceTrackingPrompt @args 
 }
 
 function Get-PerformanceHistory {
 param( [int]$count=1, [int64[]]$id=@((Get-History -count 1| Select Id).Id), [switch]$Memory )
 
 if($id.Count -eq 1) { $id = ($id[0])..($id[0]-($count-1)) } 
 
 $global:PerformanceHistory[$id]
 
 }
 
 function Format-TimeSpan($ts) {
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
 
 ##############################################################################################################
 ##############################################################################################################
 function Get-Performance {
 param(
    [Microsoft.PowerShell.Commands.HistoryInfo[]]$History = $(Get-History -count 1),
    [hashtable]$Property = @{}
 )
 ForEach($cmd in $History) {
    $msr = $null
    $avg = 7; $len = 8; $count = 1
 
    $tok = $Parser::Tokenize( $cmd.CommandLine, [ref]$null )
    if( ($tok[0].Type -eq "Number") -and 
        ($tok[0].Content -le 1) -and 
        ($tok[2].Type -eq "Number") -and 
        ($tok[1].Content -eq "..") )
    {
       $count = ([int]$tok[2].Content) - ([int]$tok[0].Content) + 1
    }
    
    $com = @( $tok | where {$_.Type -eq "Command"} | 
                      foreach { 
                         $Local:ErrorActionPreference = "SilentlyContinue"
                         get-command $_.Content 
                         $Local:ErrorActionPreference = $Script:ErrorActionPreference
                      } | 
                      where { $_.CommandType -eq "ExternalScript" } |
                      foreach { $_.Path } )
                      
    if($com.Count -gt 0){
       $msr = Get-Content -path $com | Measure-Object -Line -Word -Char
    } else {
       $msr = Measure-Object -in $cmd.CommandLine -Line -Word -Char
    }
 
    New-Object PSObject -Property (@{
          Id        = $cmd.Id
          StartTime = $cmd.StartExecutionTime
          EndTime   = $cmd.EndExecutionTime
          Duration  = Format-TimeSpan ($cmd.EndExecutionTime - $cmd.StartExecutionTime)
          Average   = Format-TimeSpan ([TimeSpan]::FromTicks( (($cmd.EndExecutionTime - $cmd.StartExecutionTime).Ticks / $count) ))
          Lines     = $msr.Lines
          Words     = $msr.Words
          Chars     = $msr.Characters
          Type      = $(if($com.Count -gt 0){"Script"}else{"Command"})
          Commmand  = $cmd.CommandLine
    } + $Property)
 }
 
 Export-ModuleMember Get-PerformanceHistory
`

