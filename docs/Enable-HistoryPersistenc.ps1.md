---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2139
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t07
---

# enable-historypersistenc - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Reloads any previously saved command history, and registers for the
 PowerShell.Exiting engine event to save new history when the shell
 exits.
 
 #>
 
 Set-StrictMode -Version Latest
 
 $GLOBAL:maximumHistoryCount = 32767
 $historyFile = (Join-Path (Split-Path $profile) "commandHistory.clixml")
 if(Test-Path $historyFile)
 {
     Import-CliXml $historyFile | Add-History
 }
 
 $null = Register-EngineEvent -SourceIdentifier `
     ([System.Management.Automation.PsEngineEvent]::Exiting) -Action {
 
     $historyFile = (Join-Path (Split-Path $profile) "commandHistory.clixml")
     $maximumHistoryCount = 1kb
 
     $oldEntries = @()
     if(Test-Path $historyFile)
     {
         $oldEntries = Import-CliXml $historyFile -ErrorAction SilentlyContinue
     }
 
     $currentEntries = Get-History -Count $maximumHistoryCount
     $additions = Compare-Object $oldEntries $currentEntries `
         -Property CommandLine | Where-Object { $_.SideIndicator -eq "=>" } |
         Foreach-Object { $_.CommandLine }
 
     $newEntries = $currentEntries | ? { $additions -contains $_.CommandLine }
 
     $history = @($oldEntries + $newEntries) |
         Sort -Unique -Descending CommandLine | Sort StartExecutionTime
 
     Remove-Item $historyFile
     $history | Select -Last 100 | Export-CliXml $historyFile
 }
`

