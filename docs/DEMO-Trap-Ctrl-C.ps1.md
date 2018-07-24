---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 542
Published Date: 2009-08-20t16
Archived Date: 2017-03-10t06
---

# demo - 

## Description

this is a demo of one way you could implement handling ctrl+c (cancelkeypressed) without using psevent or pinvoke …

## Comments



## Usage



## TODO



## function

`trap-ctrlc`

## Code

`#
 #
 function Trap-CtrlC {
    [console]::TreatControlCAsInput = $true
    if ($Host.UI.RawUI.KeyAvailable -and (3 -eq [int]$Host.UI.RawUI.ReadKey("AllowCtrlC,IncludeKeyUp,NoEcho").Character))
    {
       throw (new-object ExecutionEngineException "Ctrl+C Pressed")
    }
 }
 
 function Test-CtrlCIntercept {
    while($true) { 
       $i = ($i+1)%16
       write-host (Get-Date) -fore ([ConsoleColor]$i) -NoNewline
       foreach($sleep in 1..4) {
          Write-Host "." -fore ([ConsoleColor]$i) -NoNewline
       }
       Write-Host
    }
    
    trap [ExecutionEngineException] { 
       Write-Host "Exiting now, don't try to stop me...." -Background DarkRed
    }
 }
 
 
 
 function Test-CtrlCIntercept { 
    [console]::TreatControlCAsInput = $true
    while($true) { 
       $i = ($i+1)%16
       write-host (Get-Date) -fore ([ConsoleColor]$i)
       sleep 2; 
       if ($Host.UI.RawUI.KeyAvailable -and (3 -eq [int]$Host.UI.RawUI.ReadKey("AllowCtrlC,IncludeKeyUp,NoEcho").Character))
       {
          Write-Host "Exiting now, don't try to stop me...." -Background DarkRed
          break;
       }
    }
 }
`

