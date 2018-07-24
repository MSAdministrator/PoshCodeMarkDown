---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2418
Published Date: 
Archived Date: 2010-12-24t06
---

#  - 

## Description

journal mailboxes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Add-PSSnapIn -Name Microsoft.Exchange.Management.PowerShell.Admin -ErrorAction SilentlyContinue
 
 $databases = Get-MailboxDatabase | Where-Object {$_.Identity -notlike "sxmb201\sg01\db01"} 
 
 	foreach ($database in $databases) 
 { 
 $dbsize = (Get-MailboxStatistics -Database $database.Identity | Measure-Object -Property TotalItemSize,TotalDeletedItemSize -Sum).Sum 
 
 if (($dbsize -lt $smallestdbsize) -or ($smallestdbsize -eq $null)) 
 { 
 $smallestdbsize = $dbsize 
 $smallestdb = $database 
 } 
 } 
 
 Write-output "Enter Logon account of user to Journal"
 
 $UserToJournal = Read-Host "Mailbox to Journal"
 $RuleName = 'Journal-'+$UserToJournal
 $Password = convertto-securestring "Password123" -asplaintext -force
 $UPN = $RuleName + "@domain.pri"
 $MBX = $UserToJournal + "@domain.pri"
 $PrimarySMTP = (get-mailbox $MBX).PrimarySmtpAddress.ToString()
 
 New-Mailbox -Name $RuleName -Database $database -UserPrincipalName $UPN -FirstName $RuleName -Alias $RuleName -Password $Password -ResetPasswordOnNextLogon $False -SamAccountName $RuleName -OrganizationalUnit 'kfbdom1.kyfb.pri/Exchange/ServiceMailboxes' -DomainController sdom201.domain.pri
 
 Write-Host "Please wait for replication (10 seconds)"
 Start-Sleep -s 10 
 
 New-JournalRule -Name $UserToJournal -JournalEmailAddress $RuleName -Scope 'Global' -Enabled $true -Recipient $PrimarySMTP -DomainController sdom201.domain.pri
 
 Set-Mailbox $RuleName -HiddenFromAddressListsEnabled $true -DomainController sdom201.domain.pri
 
 
 $MailBod = @"
 The Journaling mailbox for this user can be accessed at 
 http://weboutlook.domain.com
 
 Username = $RuleName
 Password = Password123
 
 Any mail from this date going forward (Sent and Received) will be placed in the Inbox at this location. I have exported all available existing mail to a folder called PreviousData.
 
 Let me know if you have any questions or problems accessing the information.
 
 Thanks,
 Aaron Anderson
 "@
 $MailSub = "Journaled Mailbox Information"
 $MailSMTP = "kyfbsmtp.domain.pri"
 Send-MailMessage -To "aaron@domain.com" -Subject $MailSub -From "aaron@domain.com" -body $MailBod -SmtpServer $MailSMTP
 
 Export-Mailbox $MBX -TargetMailbox $RuleName -TargetFolder PreviousData -Confirm:$false
`

