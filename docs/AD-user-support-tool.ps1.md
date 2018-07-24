---
Author: ty lopes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4427
Published Date: 2013-08-30t14
Archived Date: 2013-09-03t05
---

# ad user support tool - 

## Description

#created by ty lopes

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 Import-Module ActiveDirectory
 
 $creds = Get-Credential
 
 $i = 1
 
 Do
 {
 clear-host
 
 $answers = Read-host -prompt "
         _______________________________________________________________
         Support Util
 
         a. Unlock user account
         b. change password
         c. change password and set for user to change at next logon
         d. set account to NOT change password at next logon
         e. Exit
         f. Set account to expire in XX number of days
         _______________________________________________________________
 
         Enter a b c d or e"
 
 $i++
 
 if ($answers -eq "a"){Read-Host "Enter the user account to unlock" | Unlock-ADAccount -Credential $($creds)}
 
 
 if ($answers -eq "b"){ 
     $username = Read-host -prompt "Enter Username to reset users password"
     $newPassword = (Read-Host -Prompt "Provide New Password" -AsSecureString)
     Set-ADAccountPassword -Identity $username -NewPassword $newPassword -Reset -Credential $($creds)
 }
 
 if ($answers -eq "c")
 {
     $username = Read-host -prompt "Enter Username to reset users password"
     $newPassword = (Read-Host -Prompt "Provide New Password" -AsSecureString)
     Set-ADAccountPassword -Identity $username -NewPassword $newPassword -Reset -Credential $($creds)
     Set-ADUser $username -ChangePasswordAtLogon $true -Credential $($creds)
 } 
 
 if ($answers -eq "d")
 {
     $username = Read-host -prompt "Enter Username to reset users password"
     Set-ADUser $username -ChangePasswordAtLogon $false -Credential $($creds)
 }
 
 if ($answers -eq "f")
 {
     $username = Read-host -prompt "Enter Username to extend expiration"
     $expires =  Read-host -prompt "Enter how many days away to expire the account"
     Set-adaccountExpiration $username -timespan "$expires" -Credential $($creds)
 } 
 
 if ($answers -eq "e")
 {
     $host.exit()
 }
 }
 while ($i -le 1000)
`

