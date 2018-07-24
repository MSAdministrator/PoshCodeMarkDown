---
Author: hinkle
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2085
Published Date: 2010-08-18t05
Archived Date: 2016-10-25t21
---

# exch07 snd/rec report - 

## Description

script to run daily send/receive reports in .csv format on a exchange 2007 server using power shell 1.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Start = (Get-Date -Hour 00 -Minute 00 -Second 00).AddDays(-1)
 
 $End = (Get-Date -Hour 23 -Minute 59 -Second 59).AddDays(-1)
 
 $date = get-date -Format MM-dd-yyyy
 
 $Results = @()
 
 $Sent = Get-MessageTrackingLog -Server <Server Name Ommited Needs Updated> -Start $Start -End $End -resultsize unlimited | Where { $_.EventID -eq 'Send' -or $_.EventID -eq 'Deliver' }
 
 $Received = Get-MessageTrackingLog -Server <Server Name Ommited Needs Updated> -Start $Start -End $End -resultsize unlimited | Where { $_.EventID -eq 'Receive' -or $_.EventID -eq 'TRANSFER' }
 
 $Mailboxes = Get-Mailbox
 
 $Total = $Mailboxes.Count
 $Count = 1
 
 $Mailboxes | Sort-Object DisplayName | ForEach-Object {
 	$PercentComplete = $Count / $Total * 100
 	Write-Progress -Activity "Message Tracking Log Search" -Status "Processing mailboxes" -percentComplete $PercentComplete
 
 	$Stats = "" | Select-Object Name,Sent,Received
 
 	$Email = $_.WindowsEmailAddress.ToString()
 
 	$Stats.Name = $_.DisplayName
 
 	$Stats.Sent = ($Sent | Where-Object { ($_.EventId -eq "Send" -or $_.EventID -eq "Deliver") -and ($_.Sender -eq $email) }).Count
 
 	$Stats.Received = ($Received | Where-Object { ($_.EventId -eq "RECEIVE") -and ($_.Recipients -match $email) }).Count
 
 	$Results += $Stats
 
 	$Count += 1
 }
 
 $Results | Export-CSV C:\Net_Admin_Stuff\usage_reports\send_receive_log\send_receive_log-$date.csv -NoType
`

