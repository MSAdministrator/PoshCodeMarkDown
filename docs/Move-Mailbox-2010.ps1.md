---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3577
Published Date: 2012-08-13t06
Archived Date: 2012-08-17t15
---

# move-mailbox 2010 - 

## Description

this script is called from a scheduled task running under an account with the recipient role. gets identities from a distribution group in ad, here named xc2010move but it can be anything you like. this way anyone with permissions to add users to this group can initialize a migration and with the right permissions users can even add themselves to this group (add\remove self as member).

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
 [string] $DistGroup = "XC2010Move",
 [int] $BatchCount = 5
 )
 
 Get-DistributionGroupMember $DistGroup  | get-user -Filter {Recipienttype -eq "User"} -EA SilentlyContinue | Remove-DistributionGroupMember -Identity $DistGroup -Confirm:$False
 Get-DistributionGroupMember $DistGroup | Get-Mailbox -RecipientTypeDetails "UserMailbox" -EA SilentlyContinue | Where {($_.MailboxMoveStatus -eq �Completed�)} | Set-Mailbox -IssueWarningQuota "1920MB" -ProhibitSendQuota "1984MB" -ProhibitSendReceiveQuota "2048MB"
 Get-DistributionGroupMember $DistGroup | Get-Mailbox -EA SilentlyContinue -RecipientTypeDetails "UserMailbox" | Remove-DistributionGroupMember -Identity $DistGroup -Confirm:$False
 Get-MoveRequest -MoveStatus Completed | Remove-MoveRequest -Confirm:$False
 $MB2Move = Get-DistributionGroupMember $DistGroup | Get-Mailbox -EA SilentlyContinue | Where {($_.RecipientTypeDetails -eq "LegacyMailbox") -and ($_.MailboxMoveStatus -eq �None�) -and ($_.ExchangeUserAccountControl -ne "AccountDisabled")} | Get-Random -Count $BatchCount
 $batch = "$($env:computername)_MoveMB_{0:ddMMM_yyyy}" -f (Get-Date)
 ForEach ($SingleMailbox in $MB2Move) {New-MoveRequest �Identity $SingleMailbox -BadItemLimit 100 -AcceptLargeDataLoss -Batchname $batch}
`

