---
Author: michael wulff
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6047
Published Date: 2016-10-13t13
Archived Date: 2016-05-17t12
---

# disable local users - 

## Description

disable or enable local user accounts based on csv or textfile.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $EnableUser = 512
 $DisableUser = 2
 $PasswordNotExpire = 65536
 $PasswordCantChange = 64
 $UsersToDisable = "C:\scripts\Users_To_Disable.txt"
 $users = Get-Content -path $UsersToDisable
 $computer = $env:COMPUTERNAME
 
 
 Foreach($user in $users){
  $user = [ADSI]"WinNT://$computer/$user"
  $user.userflags = $DisableUser+$PasswordNotExpire+$PasswordCantChange
  #$user.Userflags = $EnableUser+$PasswordNotExpire+$PasswordCantChange
  $user.setinfo()
 }
`

