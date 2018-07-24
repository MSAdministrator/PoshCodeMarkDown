---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 6740
Published Date: 2017-02-19t03
Archived Date: 2017-02-23t01
---

# function run-script - 

## Description

this function should be included in the powershell ise profile.ps1 and it will display the start and end times of any scripts started clicking ‘run script’ (or alt+r) in the add-ons menu; additionally they will be logged to the scripts event log (which needs creating first) and also to a text log file. this defaults to that created by the windows script monitor service (available from www.seastar.co.nf) which normally indicates the full command line used to start each script.

## Comments



## Usage



## TODO



## function

`run-script`

## Code

`#
 #
 #################################################################################
 #################################################################################
 
 function Run-Script {
  if ($host.Name -ne 'Windows PowerShell ISE Host' ) {
       return
    }
    $script = $psISE.CurrentFile.DisplayName
    if ($script.StartsWith("Untitled") -or $script.Contains("profile.") {
       return
    }
    $psISE.CurrentFile.Save()
    $logfile = "$env:programfiles\Sea Star Development\" + 
    if (!(Test-Path env:\JobCount)) {
    }
    $number = $env:JobCount.PadLeft(4,'0')
    $startTime = Get-Date -Format "dd/MM/yyyy HH:mm:ss"
    $parms = $psISE.CurrentPowerShellTab.CommandPane.Text
    $tag  = "$startTime [$script] start. --> PSE $($myInvocation.Line) $pwd\$script $parms"
    if (Test-Path $logfile) {
        $tag | Out-File $logfile -encoding 'Default' -Append
    }
    "$startTime [$script] started." 
    Write-EventLog -Logname Scripts -Source Monitor -EntryType Information -EventID 2 -Category 002 -Message "Script Job: $script (PSE$number) started."
    try {
       Invoke-Expression -Command "$pwd\$script $parms"
    }
    catch {
       Write-Host -ForegroundColor Red ">>> ERROR: $_"
    }
    $endTime = Get-Date -Format "dd/MM/yyyy HH:mm:ss"
    $tag  = "$endTime [$script] ended. --> PSE $($myInvocation.Line) $pwd\$script $parms"
    if (Test-Path $logfile) {
       $tag | Out-File $logfile -encoding 'Default' -Append
    }
    "$endTime [$script] ended."
    Write-Eventlog -Logname Scripts -Source Monitor -EntryType Information -EventID 1 -Category 002 -Message "Script Job: $script (PSE$number) ended."
    $env:JobCount = [int]$env:JobCount+1
 }
 
 $psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add("Run Script",{Run-Script}, "F2") | Out-Null
`

