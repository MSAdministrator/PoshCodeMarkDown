---
Author: jeremy d. pavleck , pavleck.net
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 738
Published Date: 
Archived Date: 2008-12-21t05
---

# prompt - battery-prompt.ps1

## Description

just a simple little script which looks at the current charge left in your battery and puts it above your prompt. adjust when it comes on with $global

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Function GLOBAL:Get-BattLevel($minLevel) {
 $charge = (Get-WmiObject -Class Win32_Battery).EstimatedChargeRemaining
 Switch($charge) {
 {($charge -ge 101) -and ($minLevel -ge 101)} {Write-Host "Batt: CHARGING" -ForeGroundColor Green}
 {($charge -ge 80) -and ($charge -le 100) -and ($charge -le $minLevel)} {Write-Host "Batt: $charge%" -ForeGroundColor Green}
 {($charge -ge 40) -and ($charge -le 79) -and ($charge -le $minLevel)} {Write-Host "Batt: $charge%" -ForeGroundColor Yellow}
 {($charge -ge 16) -and ($charge -le 39) -and ($charge -le $minLevel)} {Write-Host "Batt: $charge%" -ForeGroundColor Magenta}
 {($charge -ge 1) -and ($charge -le 15) -and ($charge -le $minLevel)} {Write-Host "Batt: $charge%" -ForeGroundColor Red}
 default {break}
 	}
  }
  
 function prompt() {
 Get-BattLevel $GLOBAL:BatteryDisplayAtPercent
 }
`

