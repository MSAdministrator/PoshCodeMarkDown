---
Author: jeffh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5483
Published Date: 2015-10-03t18
Archived Date: 2015-02-20t16
---

# check &amp; reboot appassure - 

## Description

our appassure backup service will periodically hang and stop taking scheduled snapshots. this script  runs on the appassure server as a scheduled task every 15 minutes. if the last snapshot job is older than 75 minutes (a value that takes into account the less frequent snapshots that take place over the weekend) it sends an email alert and reboots the server. ‘scriptevents’ is an event log i created on teh appassure server to log these automatic reboots.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $LastEvent = Get-EventLog AppAssure -newest 1
 $TimeCheck = Get-EventLog AppAssure -After (Get-Date).AddMinutes(-75)
 If ($TimeCheck -eq $null)
 {
 $EmailFrom = "jeff@example.com"
 $EmailTo = "jeff@example.com"
 $Subject = "AppAssure has hung - last event was at " + $LastEvent.TimeGenerated
 $Body = "AppAssure has hung - last event was at " + $LastEvent.TimeGenerated
 $SMTPServer = "mail.example.com"
 $SMTPClient = New-Object Net.Mail.SmtpClient($SmtpServer, 25)
 $SMTPClient.Send($EmailFrom, $EmailTo, $Subject, $Body)
 Write-EventLog -LogName ScriptEvents -Source AppAssureHung -EventId 99 -Message $Body
 Restart-Computer -Force
 }
`

