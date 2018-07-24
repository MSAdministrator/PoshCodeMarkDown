---
Author: themoblin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6627
Published Date: 2017-11-16t20
Archived Date: 2017-03-15t18
---

# mailboxfolderpermissions - 

## Description

enumerates mailbox folder permissions for all folders in all mailboxes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $mailboxes = get-mailbox
 
 $mailboxes| foreach-object {
 	
 	$alias = $_.alias
 	$folders = get-mailboxfolderstatistics $_
 	$foldernames = $folders|select-object name
 
 	"--------------------------------------------------------------" | Out-File C:\MailboxPermissions.txt -append
 	"" | Out-File C:\MailboxPermissions.txt -append
 	 "Processing permissions on $alias" | Out-File C:\MailboxPermissions.txt -append
 	 "" | Out-File C:\MailboxPermissions.txt -append
 	$foldernames | foreach-object {
 
 		$concat = $alias + ":\" + $_.name
 		get-mailboxfolderpermission -identity $concat -erroraction silentlycontinue | ft foldername,User,AccessRights | Out-File C:\MailboxPermissions.txt -append
 				      }
 	 "" | Out-File C:\MailboxPermissions.txt -append
 	 "--------------------------------------------------------------" | Out-File C:\MailboxPermissions.txt -append
 			   }
`

