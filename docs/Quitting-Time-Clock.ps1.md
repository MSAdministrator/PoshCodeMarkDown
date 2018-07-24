---
Author: dan in philly
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6873
Published Date: 2017-05-01t16
Archived Date: 2017-05-13t17
---

# quitting time clock - 

## Description

demonstrates how to resize a powershell window, use a count-down timer, toggle a keystroke, and automate logging off your computer.  i wrote this so i’d have a simple reminder of these things but i ended up running the script at the start of each work day.  i’ve also distributed the “toggle scroll lock” part to a lot of colleagues who got sick of the corporate screen saver gpo.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Clear
     $Host.UI.RawUI.WindowTitle = "Quitting Time Clock"
     $Size = $Host.UI.RawUI.WindowSize
     $Size.Width = 30
     $Size.Height = 3
     $Host.UI.RawUI.WindowSize = $Size
 Write-Host "What is today's quitting time?"
 $qt = Read-Host "(HH:mm:ss)"
 
 Do {Clear
 $tm = Get-Date -Format HH:mm:ss
 
 If($tm.Substring(4,4) -eq "0:00" -or $tm.Substring(4,4) -eq "5:00"){
     $shell = New-Object -com "Wscript.Shell"
     $shell.sendkeys(�{ScrollLock}{ScrollLock}�)
     $shell.Dispose}
 
     $ts = New-Timespan $(get-date) $qt
     Write-Host $([string]::Format("Time Remaining: {0:d2}:{1:d2}:{2:d2}",
     $ts.hours, $ts.minutes, $ts.seconds)) -ForegroundColor Cyan
 
     Write-Host "         " -NoNewline
     Write-Host $tm -ForegroundColor Yellow
     sleep 1 }
 Until ($tm -ge $qt)
 
 $LOTime = (Get-Date).AddMinutes(5).ToString("HH:mm:ss")
 Do {Clear
     $LOCount = New-TimeSpan $(Get-Date) $LOTime
     Write-Host $([string]::Format("Logoff in: {0:d2}:{1:d2}:{2:d2}",
     $LOCount.Hours, $LOCount.Minutes, $LOCount.Seconds)) -ForegroundColor Red
     Sleep 1}
 Until ($LOCount.Minutes -eq 0 -and $LOCount.Seconds -eq 0)
 Shutdown /L /F
`

