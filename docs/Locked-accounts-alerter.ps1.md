---
Author: ty lopes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5077
Published Date: 2016-04-14t13
Archived Date: 2016-10-18t06
---

# locked accounts alerter - 

## Description

edit

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 
 
 
 	start-sleep 10
 
 	$dcName = "$env:computername.$env:userdnsdomain"
 	$eventID = "4740"
 	$mailServer = "smtpServer"
 	$eSubject = "AD account locked"
 	$emailAddy = "user@domain.com"
 
 	$lockEvent = get-eventlog -logname security -computername $dcName -instanceid $eventID -newest 1
 
 	$emailBody = $lockEvent.message
 	Send-MailMessage �From lockedAccount@domain.com �To $emailAddy �Subject $eSubject �Body $emailBody �SmtpServer $mailServer
 
`

