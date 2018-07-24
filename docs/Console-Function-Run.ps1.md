---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3087
Published Date: 2012-12-09t23
Archived Date: 2012-01-14t07
---

# console function run - 

## Description

this is the console version of the ise run-script function posted earlier. typing ‘run example’ from the command line will run and log the start and end times of ‘example.ps1’

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Run ([String]$scriptName = '-BLANK-') {
    session (from $pwd) in the Scripts Event Log.
    It should be placed in the Console $profile. Scripts should be started
    by typing 'Run example' to capture example.ps1, for example. 
    The default logfile is that used by the Windows Script Monitor Service, 
    available from www.SeaStarDevelopment.Bravehost.com
 #>   
     if ($host.Name -ne 'ConsoleHost') {
        return
     }
     $logfile = "$env:programfiles\Sea Star Development\" + 
         "Script Monitor Service\ScriptMon.txt"
     $parms  = $myInvocation.Line -replace "run(\s+)$scriptName(\s*)"
     $script = $script + ".ps1"
     if (Test-Path $pwd\$script) {
         if(!(Test-Path Variable:\Session.Script.Job)) {
             Set-Variable Session.Script.Job -value 1 -scope global `
                 -description "Script counter"
         }
         $Job    = Get-Variable -name Session.Script.Job
         $number = $job.Value.ToString().PadLeft(4,'0')
         $startTime = Get-Date -Format "dd/MM/yyyy HH:mm:ss"
         $tag  = "$startTime [$script] start. --> $($myInvocation.Line)"
         if (Test-Path $logfile) {
             $tag | Out-File $logfile -encoding 'Default' -Append
         }
         Write-EventLog -Logname Scripts -Source Monitor -EntryType Information -EventID 2 -Category 002 -Message "Script Job: $script (PS$number) started."
         Invoke-Expression -command "$pwd\$script $parms"
         $endTime = Get-Date -Format "dd/MM/yyyy HH:mm:ss"
         $tag  = "$endTime [$script] ended. --> $($myInvocation.Line)"
         if (Test-Path $logfile) {
             $tag | Out-File $logfile -encoding 'Default' -Append
         }
         Write-Eventlog -Logname Scripts -Source Monitor -EntryType Information -EventID 1 -Category 001 -Message "Script Job: $script (PS$number) ended."
         $job.Value += 1 
     }
     else {
         Write-Error "$pwd\$script does not exist."
     }
 }
`

