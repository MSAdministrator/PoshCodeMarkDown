---
Author: ermias
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5023
Published Date: 2014-03-26t13
Archived Date: 2014-03-30t06
---

# dir.mus@com.net - 

## Description

add new smtp address from csv and set new address primary

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 import-csv .\source.csv | foreach {
 $user = Get-Mailbox $_.alias
 $user.emailAddresses+= $_.addnewemailaddress
 $user.primarysmtpaddress = $_.addnewemailaddress
 Set-Mailbox $user -emailAddresses $user.emailAddresses
 set-Mailbox $user -PrimarySmtpAddress $user.primarysmtpaddress
 }
`

