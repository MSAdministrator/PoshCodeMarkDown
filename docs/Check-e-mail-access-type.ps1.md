---
Author: osnilton k m
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3924
Published Date: 2013-01-31t10
Archived Date: 2013-02-04t05
---

# check e-mail access type - 

## Description

want to know what type of access a user has to exchange server?

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $ErrorActionPreference = "silentlycontinue"
 
 $login = read-host -prompt "Type the user login"
 
 $Status = @( Get-ADuser $login | select SamAccountName).count 
 
 If($Status -eq 0) {
 
 Write-Host No such user exists! -FOREGROUNDCOLOR RED
 
 ./the_script_name.ps1
 
 } Else {Write-Host Working on it! -FOREGROUNDCOLOR GREEN
 
  
 }
 
 
 Get-Mailbox $login | Get-CASMailbox
`

