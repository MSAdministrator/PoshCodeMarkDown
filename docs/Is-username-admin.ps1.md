---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4563
Published Date: 2013-10-27t08
Archived Date: 2013-11-03t20
---

# is {username} admin? - 

## Description

three ways (that i personally know) how to check if user has admin rights.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 "Is {0} admin? {1}`n" -f $env:username, [bool](New-Object -com CompatUI.Util).CheckAdminPrivileges()
 
 $usr = [Security.Principal.WindowsIdentity]::GetCurrent()
 $res = (New-Object Security.Principal.WindowsPrincipal $usr).IsInRole(
   [Security.Principal.WindowsBuiltInRole]::Administrator
 )
 "Is {0} admin? {1}`n" -f $usr.Name.Split("\")[1], $res
 
 [xml]$loc = @'
 <Culture>
   <Language en="Administrators"
             de="Administratoren"
 </Culture>
 '@
 
 $d = $env:userdomain + '/' + $loc.Culture.Language.((Get-Culture).TwoLetterISOLanguageName)
 $res = [bool]@(([adsi]"WinNT://$d").PSBase.Invoke("Members")).Length
 "Is {0} admin? {1}`n" -f $env:username, $res
`

