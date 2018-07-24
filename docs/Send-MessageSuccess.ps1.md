---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2417
Published Date: 
Archived Date: 2010-12-24t06
---

#  - 

## Description

evac mailboxes

## Comments



## Usage



## TODO



## script

`send-messagesuccess`

## Code

`#
 #
 Offline defrags are not a good option when dealing with large databases that can take 10 hours to copy around. 
 I can't have 300 users offline for that long. Specifiy the DB you want to emtpy with the $EvacDB variable. The script will find the smallest 
 database and then move 2 random mailboxes to that location. It will determine the smallest database after each pair of mailboxes is moved and
 maintain that databases are not filled beyond 80gb. This is useful for a number of reasons. If you have 100GB of mailboxes to move, you may not have 
 that much free space in any one DB. #>
 
 function Send-MessageSuccess {
 Send-MailMessage -To aaron@domain.com -From aaron@domain.com -Subject "Move-Mailbox COMPLETE" -Body "All mailboxes have been evacuated." -SmtpServer smtp201;
 }
 function Send-MessageFailure {
 Send-MailMessage -To aaron@domain.com -From aaron@domain.com -Subject "Move-Mailbox FAILURE" -Body "There is a problem with the script." -SmtpServer smtp201;
 }
 
 $databases = Get-MailboxDatabase | Where-Object {$_.Identity -notlike "sxmb201\sg01\db01"} 
 
 $EvacDB = "SXMB201\SG04\DB04"
 
  do {
 	   
 	foreach ($database in $databases) 
 { 
 $dbsize = (Get-MailboxStatistics -Database $database.Identity | Measure-Object -Property TotalItemSize,TotalDeletedItemSize -Sum).Sum 
 
 if (($dbsize -lt $smallestdbsize) -or ($smallestdbsize -eq $null)) 
 { 
 $smallestdbsize = $dbsize 
 $smallestdb = $database 
 } 
 } 
  
 $SmallestDBSize = ((Get-MailboxStatistics -database $smallestdb | Measure-Object -Property TotalItemSize,TotalDeletedItemSize -Sum | Select-Object Sum |Measure-Object -Property Sum -Sum).Sum.ToString() /1mb)
 
 if ($SmallestDBSize -lt 81920) {Write-Host "There is enough space, script will proceed"}
 else {Write-Host "There is not enough space, script will stop";break}
 	   
 if ((Get-Mailbox -Database $EvacDB) -eq $null) {Write-Host "No more mailboxes left to move, take a nap.";Send-MessageSuccess;break} 
 else {Get-Mailbox -Database $EvacDB | Get-Random -Count 2 | Move-Mailbox -TargetDatabase $smallestdb -Confirm:$false}
 
 } while ((Get-Mailbox -Database $EvacDB) -ne $null)
`

