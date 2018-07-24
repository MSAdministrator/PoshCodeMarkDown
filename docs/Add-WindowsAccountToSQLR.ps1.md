---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4474
Published Date: 2013-09-17t18
Archived Date: 2016-10-18t11
---

# add-windowsaccounttosqlr - add-windowsaccounttosqlrole.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`add-windowsaccounttosqlrole`

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 
 Function Add-WindowsAccountToSQLRole ([String]$Server, [String] $User, [String]$Role)
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
                     $svrole = $svr.Roles | where {$_.Name -eq $Role}
                     $svrole.AddMember("$LoginName")
                     }
                 }
 
             else
                 {
                 $SqlUser = New-Object -TypeName Microsoft.SqlServer.Management.Smo.Login $Server, $User
                 $SqlUser.LoginType = 'WindowsUser'
                 $SqlUser.Create()
                 $LoginName = $SQLUser.Name
                 if($Role -notcontains "public")
                     {
                     $svrole = $svr.Roles | where {$_.Name -eq $Role}
                     $svrole.AddMember("$LoginName")
                     }
                 }
         }
 
 }
`

