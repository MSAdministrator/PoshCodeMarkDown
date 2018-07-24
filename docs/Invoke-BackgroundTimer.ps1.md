---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5560
Published Date: 2015-10-31t21
Archived Date: 2015-01-31t20
---

# invoke-backgroundtimer - 

## Description

this is an example of how to use timers, events, and jobs together in a way that doesn’t block the event, but does block the host. see the example for … an example. more later when i have time to think of them ;-)

## Comments



## Usage



## TODO



## function

`invoke-backgroundtimer`

## Code

`#
 #
 function Invoke-BackgroundTimer {
     #.Synopsis
     #.Example
     [CmdletBinding(DefaultParameterSetName="Milliseconds")]
     param(
         [Parameter(Position=0,Mandatory=$True)]
         [ScriptBlock[]]$Action,
 
         [Parameter(ParameterSetName="Milliseconds")]
         [int]$TimerMilliseconds = 1000,
         [Parameter(ParameterSetName="Milliseconds")]
         [int]$LimitSeconds = "30",
 
         [Parameter(ParameterSetName="TimeSpan")]
         [TimeSpan]$Every = "0:0:1",
         [Parameter(ParameterSetName="TimeSpan", Mandatory=$true)]
         [TimeSpan]$For
 
     )
 
     if($PSCmdlet.ParameterSetName -eq "Timespan") {
         $TimerMilliseconds = $Every.Milliseconds
         $LimitMilliseconds = $For.Milliseconds
     } else {
         $LimitMilliseconds = $LimitSeconds * 1000
     }
 
     $ProcessTimer = New-Object System.Timers.Timer $TimerMilliseconds
 
     $StopWatch = New-Object System.Diagnostics.StopWatch
 
     $MessageData = @{
         Action = $Action
         StopWatch = $StopWatch
         Limit = $LimitMilliseconds
     }
 
     $Job = Register-ObjectEvent $ProcessTimer -SourceIdentifier LoopTimer -EventName Elapsed -MessageData $MessageData -Action {
         if($Event.MessageData.StopWatch.ElapsedMilliseconds -ge $Event.MessageData.Limit) {
             Unregister-Event LoopTimer
             break
         }
 
         Write-Progress "Processing" -SecondsRemaining (($Event.MessageData.Limit - $Event.MessageData.StopWatch.ElapsedMilliseconds) / 1000)
 
         $Event.MessageData.Action | %{ & $_ }
     }
 
     $ProcessTimer.Start()
     $StopWatch.Start()
 
     Write-Warning "If you stop the script using Ctrl+C at this point, you need to manually Unregister-Event LoopTimer"
     while(!$Job.Finished.WaitOne($TimerMilliseconds/10)) {
         $Job | Receive-Job
     }
     $Job | Receive-Job
     $Job | Remove-Job
 
     $ProcessTimer.Dispose()
     $StopWatch.Stop()
 }
`

