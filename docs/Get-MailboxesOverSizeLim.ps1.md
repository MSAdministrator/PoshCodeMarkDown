---
Author: chris brown
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2979
Published Date: 2012-09-29t14
Archived Date: 2012-01-14t07
---

# get-mailboxesoversizelim - 

## Description

emails a report of exchange 2010 mailboxes over their size limit.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 
 $SmtpServer = "mailserver.yourdomain.local"
 $FromAddress = "alerts@yourdomain.com"
 $ToAddress = "HelpDesk@yourdomain.com"
 $Subject = "[Exchange] Mailboxes over size limits"
 #
 #
 
 if ( (Get-PSSnapin -Name Microsoft.Exchange.Management.PowerShell.E2010 -ErrorAction SilentlyContinue) -eq $null) {
 	Write-Verbose "Exchange 2010 snapin is not loaded. Loading it now."
 	try { Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010; Write-Verbose "Loaded Exchange 2010 snapin" }
 	catch { Write-Error "Could not load Exchange 2010 snapins!"; }
 }
 
 $overLimit = Get-Mailbox -ResultSize unlimited| Get-MailboxStatistics -WarningAction SilentlyContinue | Where {"ProhibitSend","MailboxDisabled" -contains $_.StorageLimitStatus}
 
 if ($overLimit.Count -gt 0) {
 	$mailBody = $overLimit | ft DisplayName,TotalItemSize,StorageLimitStatus | Out-String
 	Send-MailMessage -Body $mailBody.ToString() -From $FromAddress -SmtpServer $SmtpServer -Subject $Subject -To $toAddress
 } else {
 	"No results"
 }
`

