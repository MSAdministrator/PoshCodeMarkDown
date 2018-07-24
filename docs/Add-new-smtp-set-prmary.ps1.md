---
Author: ermias
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6825
Published Date: 2017-03-29t14
Archived Date: 2017-04-01t14
---

# add new smtp/set prmary - 

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

