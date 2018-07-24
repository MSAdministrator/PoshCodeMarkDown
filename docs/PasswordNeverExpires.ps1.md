---
Author: vulcanx
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2998
Published Date: 2012-10-12t02
Archived Date: 2012-01-14t07
---

# passwordneverexpires - 

## Description

email list of ad accounts with passwordneverexpires set

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 
 Import-Module ActiveDirectory
 
 Add-PSSnapin Quest.ActiveRoles.ADManagement
 
 $currentDate = get-date -format "yyyy-MM-dd"
 
 Get-Qaduser -PasswordNeverExpires | Out-File "C:\Password_Exp\Exp_$currentdate.txt"
 
 $attach = "C:\Password_Exp\Exp_$currentdate.txt"
 
 Send-MailMessage -From 'email1@domain.com' -to 'email2@domain.com' -Subject `
 "Password Expiration - $currentDate" -Body "User Accounts whose Passwords Never Expire `n `r Please refer to the txt attachment" `
 -Attachments $attach -SmtpServer 'smtp01.domain.com'
`

