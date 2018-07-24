---
Author: zespri
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5273
Published Date: 2014-07-02t01
Archived Date: 2014-08-29t21
---

# is local admin - 

## Description

i could not find a solution that checks if a user is in local admin groups that can handle a situation if the user is there indirectly, that is a member of a group that is a part of the admin group. the below is what i came up with. the $env

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $accountName = "BLA\user"
 
 add-type -AssemblyName System.DirectoryServices.AccountManagement
 $pc = new-object System.DirectoryServices.AccountManagement.PrincipalContext Domain,$env:USERDOMAIN
 $p = [System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($pc,$accountName)
 
 $wi = new-object System.Security.Principal.WindowsIdentity $p.UserPrincipalName
 $wp = [System.Security.Principal.WindowsPrincipal]$wi
 $wp.IsInRole("Administrators")
`

