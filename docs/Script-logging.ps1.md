---
Author: shauncroucher
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5269
Published Date: 2014-06-30t13
Archived Date: 2014-10-21t19
---

# script logging - 

## Description

add to top of your scripts and the script will automatically create a log file called <script name>.log to the appdata folder

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $ScriptName = $MyInvocation.mycommand.name 
 $LocalAppDir = "$(gc env:LOCALAPPDATA)\PS_Data" 
 $LogName = $ScriptName.replace(".ps1", ".log") 
 
 trap
 [Exception] { 
 sendl
 "error: $($_.Exception.GetType().Name) - $($_.Exception.Message)" 
 } 
 function LogFileCheck 
 {
 {
 mkdir $LocalAppDir 
 New-Item "$LocalAppDir\$LogName" -type file 
 break 
 }
 if
 {
 $NewLogFile = $LogName.replace(".log", " ARCHIVED $(Get-Date -Format dd-MM-yyy-hh-mm-ss).log") 
 ren "$LocalAppDir\$LogName" "$LocalAppDir\$NewLogFile" 
 }
 }
 }
 {
 $toOutput
 = "$(get-date) > $message " | Out-File "$LocalAppDir\$LogName" -append -NoClobber 
 }
 LogFileCheck
`

