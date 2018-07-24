---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 965
Published Date: 
Archived Date: 2009-03-27t18
---

# pscron - 

## Description

cron daemon for powershell

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $cronfile = "C:\temp\cron.csv"
 $logfile = "C:\temp\pscron.log"
 $runtime = $TRUE
 $script:crontable = $FALSE
 $script:log = ""
 
 function initCron {
 	if (!(Test-Path $cronfile)) { 
 		Write-Host "Please create a crontab file!"
 		break
 	}
 	$script:crontable = Import-Csv $cronfile
 	if (!$script:crontable) {
 		break
 	}
 }
 
 function splitDate {
 	$crontemp = @()
 	foreach ($cronentry in $script:crontable) {
 		foreach ($min in $cronentry.min.split(" ")) {
 			foreach ($hour in $cronentry.hour.split(" ")) {
 				foreach ($day in $cronentry.day.split(" ")) {
 					foreach ($month in $cronentry.month.split(" ")) {
 						$row = "" | Select-Object min,hour,day,month,command,ps
 						$row.min = $min
 						$row.hour = $hour
 						$row.day = $day
 						$row.month = $month
 						$row.command = $cronentry.command
 						$row.ps = $cronentry.ps
 						$crontemp += $row
 					}
 				}
 			}
 		}
 	}
 	$script:crontable = $crontemp
 }
 
 function writeLog {
 	$logcontainer = "Executed on - "
 	$logcontainer += Get-Date
 	$logcontainer += "`n"
 	$logcontainer += $script:log
 	$logcontainer += "`n"
 	
 	$logcontainer | Out-File $logfile -Append 
 }
 
 function showLog {
 	if (!(Test-Path $logfile)) {
 		Write-Host "No Log File Available"
 		break
 	}
 	
 	$logcontent = Get-Content $logfile
 	$logcontent | more
 }
 
 function showCrontable {
 	$croncontent = Import-Csv $cronfile
 	$croncontent | ft | more
 }
 
 function clearLog {
 	$empty = ""
 	$empty | Out-File $logfile
 }
 
 function executeCron {
 	$date = Get-Date
 	foreach ($cronentry in $script:crontable) {
 		if (($cronentry.month -eq $date.month) -or ($cronentry.month -eq "*")) {
 			if (($cronentry.day -eq $date.day) -or ($cronentry.day -eq "*")) {
 				if (($cronentry.hour -eq $date.hour) -or ($cronentry.hour -eq "*")) {
 					if (($cronentry.min -eq $date.minute) -or ($cronentry.min -eq "*")) {
 						if ($cronentry.ps -eq "1") {
 							$script:log = $cronentry.command
 							Invoke-Expression $($cronentry.command)
 						} elseif (!$cronentry.ps) {
 							$log = [Diagnostics.Process]::Start($cronentry.command)
 							$script:log = $log.ProcessName
 						}
 						$script:log
 					}
 				}
 			}
 		}
 	}
 }
 
 
 function initConsole {
 	do {
 		$cmd = Read-Host -Prompt "PoSh Cron >"
 		
 		switch ($cmd) {
 		
 			showlog {
 				showLog
 			}
 			
 			showcrontable{
 				showCrontable
 			}
 			
 			clearlog {
 				clearLog
 			}
 			
 			help {
 				Write-Host "showlog - Display Log File"
 				Write-Host "showcrontable - Display current crontable"
 				Write-Host "clearlog - Clear Log File"
 			}
 
 			quit { }
 			default {
 				Write-Host "Unknown command"
 			}
 		}
 		
 	} until ($cmd -eq "quit")
 }
 
 
 
 while ($runtime) {
 	initCron
 	splitDate
 	executeCron
 	writelog
 	for ($i=0;$i -lt 59;$i++) {
 		if ($Host.UI.rawui.keyavailable) {
 			$key = $Host.UI.rawui.Readkey("NoEcho,IncludeKeyUp")
 		}
 
 		if ($key.VirtualKeyCode -eq 67) {
 			initConsole
 			$key = ""
 		}
 		
 		if ($key.VirtualKeyCode -eq 27) {
 			exit
 		}
 		sleep 1
 	}
 } 
 
`

