---
Author: ermias
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6038
Published Date: 2016-10-05t14
Archived Date: 2016-05-17t16
---

# @teste.com.br - 

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

