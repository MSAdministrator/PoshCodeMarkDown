---
Author: justin grote
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4633
Published Date: 2015-11-22t21
Archived Date: 2015-10-23t21
---

# send-emailnotifyoflogin - 

## Description

sends an email on event trigger id 21 and 23 (logon and logoff). useful for auditing access to a server.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 
 
 
 
 if (!$PSEmailServer) {$PSEmailServer = "mail"}
 $logaction = $args[0]
 $msgRecipient = $args[1]
 
 
 $ComputerName = [System.Net.Dns]::GetHostByName(($env:computerName)).HostName.ToLower()
 
 switch ($logaction) {
     logon {$ActionEventID = 21}
     logoff {$ActionEventID = 23}
 }
 
 $EventToSend = get-winevent -maxevents 1 -filterhashtable  @{ LogName = "Microsoft-Windows-TerminalServices-LocalSessionManager/Operational";ID = $ActionEventID }
 
 
 $targetUser = ($EventToSend.message.split("`n") | select-string "User:").ToString().substring(6).Normalize()
 
 $targetuser = $targetuser.substring(0,$targetuser.length-1)
 
 $msgSubject = ("Notification: $targetuser $logaction - $computername").Replace('`r', ' ').Replace('`n', ' ')
 
 $msgBody = "$($EventToSend.TimeCreated) 
 $($eventToSend.message.Replace('`n', '`r`n'))"
 
 send-mailmessage -To $msgRecipient -Subject $msgSubject -Body $msgBody -From "scheduledtask@$computername"
 
`

