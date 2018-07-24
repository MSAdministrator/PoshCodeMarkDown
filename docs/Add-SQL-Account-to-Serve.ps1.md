---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4475
Published Date: 2014-09-17t18
Archived Date: 2014-08-18t21
---

# add sql account to serve - add-sqlaccounttosqlrole.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`add-sqlaccounttosqlrole`

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 
 Function Add-SQLAccountToSQLRole ([String]$Server, [String] $User, [String]$Password, [String]$Role)
 {
 
 $Svr = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
 
 $SVRRole = $svr.Roles[$Role]
     if($SVRRole -eq $null)
         {
         Write-Host " $Role is not a valid Role on $Server"
         }
 
     else
         {
     		if($svr.Logins.Contains($User))
 			    {
                 $SqlUser = New-Object -TypeName Microsoft.SqlServer.Management.Smo.Login $Server, $User
                 $LoginName = $SQLUser.Name
                 if($Role -notcontains "public")
                     {
                     
                     $SVRRole.AddMember($LoginName)
                     }
                 }
 
             else
                 {
                 $SqlUser = New-Object -TypeName Microsoft.SqlServer.Management.Smo.Login $Server, $User
                 $SqlUser.LoginType = 'SqlLogin'
                 $SqlUser.PasswordExpirationEnabled = $false
                 $SqlUser.Create($Password)
                 $LoginName = $SQLUser.Name
                 if($Role -notcontains "public")
                     {
                     $SVRRole.AddMember($LoginName)
                     }
                 }
         }
 
 }
`

