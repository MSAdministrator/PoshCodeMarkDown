---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6563
Published Date: 
Archived Date: 2016-11-28t14
---

#  - 

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

