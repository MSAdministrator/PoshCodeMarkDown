---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2394
Published Date: 
Archived Date: 2010-12-03t12
---

# bulk change ad passwords - 

## Description

bulk change account passwords without requiring elevated permissions.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 #-------------------------------------------------------------
  Add-PSSnapin Quest.ActiveRoles.ADManagement
 
 
 
 
 $userlist | foreach-object {
     Write-output -----------------------------------------------
     Write-output $_.NTAccountName
 
     $ADUser= Get-QADUser  $_.NTAccountName
     $ADSIUser = [adsi] $ADUser.Path
     
     Write-output $ADSIUser.displayName
     Write-output "Changing password from $($_.OldPassword) to $($_.NewPassword) ...."
     $result = $ADSIUser.psbase.invoke("ChangePassword",$_.OldPassword, $_.NewPassword)
     Write-output "Password change result $result"
 }
`

