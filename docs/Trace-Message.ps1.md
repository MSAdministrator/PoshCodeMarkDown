---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.25
Encoding: ascii
License: cc0
PoshCode ID: 6457
Published Date: 2016-07-26t05
Archived Date: 2016-07-29t20
---

# trace-message - 

## Description

a trace/logging system that wraps write-verbose, write-debug or write-warning to add timestamps and invocation source to the output, as well as allowing complicated messages to be calculated only when the output will actually be displayed.

## Comments



## Usage



## TODO



## function

`trace-message`

## Code

`#
 #
 function Trace-Message {
     <#
         .Synopsis
             Wrap Verbose, Debug, or Warning output with command profiling trace showing script line and time elapsed
         .Description
 
             Creates a stopwatch that tracks the time elapsed while a script runs, and adds caller position and time to the output
         .Example
             foreach($i in 1..20) { sleep -m 50; Trace-Message "Progress $i" }
 
             Demonstrates the simplest use of Trace-Message to add a duration timestamp to the message.
         .Example
             function Test-Trace {
                 [CmdletBinding()]param()
                 foreach($i in 1..20) {
                     $i
                     Trace-Message {
                         $ps = (Get-Process | sort PM -Desc | select -first 2) 
                         "Memory hog {1} using {0:N2}GB more than next process" -f (($ps[0].WS -$ps[1].WS) / 1GB), $ps[0].Name
                     } @PSBoundParameters
                 }
             }
 
             Demonstrates how using a scriptblock can avoid calculation of complicated output when -Verbose is not set.  In this example, "Test-Trace" by itself will output 1-20 in under 20 miliseconds, but with verbose output, it can take over 1.25 seconds
     #>  
     [CmdletBinding(DefaultParameterSetName="Precalculated")]
     param(
         [Parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,ParameterSetName="Precalculated",Position=0)]
         [Parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,ParameterSetName="PrecalculatedWarningOutput",Position=0)]
         [Parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,ParameterSetName="PrecalculatedDebugOutput",Position=0)]
         [PSObject]$Message,
 
         [Parameter(Mandatory=$true,ParameterSetName="DeferredTemplate",Position=0)]
         [Parameter(Mandatory=$true,ParameterSetName="DeferredTemplateWarningOutput",Position=0)]
         [Parameter(Mandatory=$true,ParameterSetName="DeferredTemplateDebugOutput",Position=0)]
         [ScriptBlock]$DeferredTemplate,
 
         [Parameter(Mandatory=$true,ParameterSetName="PrecalculatedWarningOutput")]
         [Parameter(Mandatory=$true,ParameterSetName="DeferredTemplateWarningOutput")]
         [Alias("AsWarning")]
         [switch]$WarningOutput,
 
         [Parameter(Mandatory=$true,ParameterSetName="PrecalculatedDebugOutput")]
         [Parameter(Mandatory=$true,ParameterSetName="DeferredTemplateDebugOutput")]
         [switch]$DebugOutput,
 
         [switch]$ResetTimer,
 
         [switch]$KillTimer,
 
         [string]$ElapsedFormat,
 
         [Diagnostics.Stopwatch]$Stopwatch
     )
     begin {
         if($Stopwatch) {
             ${Script:Trace Message Timer} = $Stopwatch    
             ${Script:Trace Message Timer}.Start()
         }
         if(-not ${Trace Message Timer}) {
             ${global:Trace Message Timer} = New-Object System.Diagnostics.Stopwatch
             ${global:Trace Message Timer}.Start()
 
             $PreTraceTimerPrompt = $function:prompt
 
             $function:prompt = { 
                 if(${global:Trace Message Timer}) {
                     ${global:Trace Message Timer}.Stop()
                     Remove-Variable "Trace Message Timer" -Scope global -ErrorAction SilentlyContinue
                 }
                 & $PreTraceTimerPrompt
                 ${function:global:prompt} = $PreTraceTimerPrompt
             }.GetNewClosure()
         }
 
         if($ResetTimer -or -not ${Trace Message Timer}.IsRunning)
         {
             ${Trace Message Timer}.Restart()
         }
 
         $w = $Host.UI.RawUi.BufferSize.Width
     }
 
     process {
         if(($WarningOutput -and $WarningPreference -eq "SilentlyContinue") -or
            ($DebugOutput -and $DebugPreference -eq "SilentlyContinue") -or
            ($VerbosePreference -eq "SilentlyContinue")) { return }
 
         [string]$Message = if($DeferredTemplate) { 
                              ($DeferredTemplate.InvokeReturnAsIs(@()) | Out-String -Stream) -join "`n"
                            } else { "$Message" }
        
         $Message = $Message.Trim()
         
         $Location = if($MyInvocation.ScriptName) {
                         $Name = Split-Path $MyInvocation.ScriptName -Leaf
                         "${Name}:" + "$($MyInvocation.ScriptLineNumber)".PadRight(4)
                     } else { "" }
         
         $Tail = $(if($ElapsedFormat) {
                       "{0:$ElapsedFormat}" -f ${Trace Message Timer}.Elapsed
                   }
                   elseif(${Trace Message Timer}.Elapsed.TotalHours -ge 1.0) {
                       "{0:h\:mm\:ss\.ffff}" -f ${Trace Message Timer}.Elapsed
                   }
                   elseif(${Trace Message Timer}.Elapsed.TotaMinutes -ge 1.0) {
                       "{0:mm\m\ ss\.ffff\s}" -f ${Trace Message Timer}.Elapsed
                   }                    
                   else {
                       "{0:ss\.ffff\s}" -f ${Trace Message Timer}.Elapsed
                   }).PadLeft(12)
 
         $Tail = $Location + $Tail 
 
         $Length = ($Message.Length + 10 + $Tail.Length)
         $PaddedLength = if($Length -gt $w -and $w -gt (25 + $Tail.Length)) {
                             [string[]]$words = -split $message
                             $lines = 0
                             do {
                                 do {
                                     $short += 1 + $words[$count++].Length
                                 } while (($words.Count -gt $count) -and ($short + $words[$count].Length) -lt $w)
                                 $Lines++
                                 if(($Message.Length + $Tail.Length) -gt ($w * $lines)) {
                                     $short = 0
                                 }
                             } while($short -eq 0)
                             $Message.Length + ($w - $short) - $Tail.Length
                         } else { 
                             $w - 10 - $Tail.Length
                         }
 
         $Message = "$Message ".PadRight($PaddedLength, "$([char]8331)") + $Tail
 
         if($WarningOutput) {
             Write-Warning $Message
         } elseif($DebugOutput) {
             Write-Debug $Message
         } else {
             Write-Verbose $Message
         }
     }
 
     end {
         if($KillTimer -and ${Trace Message Timer}) {
             ${Trace Message Timer}.Stop()
             Remove-Variable "Trace Message Timer" -Scope Script -ErrorAction Ignore
             Remove-Variable "Trace Message Timer" -Scope Global -ErrorAction Ignore
         }
     }
 }
`

