---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6765
Published Date: 2017-03-02t12
Archived Date: 2017-03-12t06
---

# add user to database rol - add-usertorole.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`add-usertorole`

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 
 Function Add-UserToRole ([string] $server, [String] $Database , [string]$User, [string]$Role)
 {
 $Svr = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
     if($svr.Databases.name -notcontains $Database)
         {
         Write-Host " $Database is not a valid database on $Server"
         Write-Host " Databases on $Server are :"
         $svr.Databases|select name
         break
         }
         $db = $svr.Databases[$Database]
     if($db.Roles.name -notcontains $Role)
         {
         Write-Host " $Role is not a valid Role on $Database on $Server  "
         Write-Host " Roles on $Database are:"
         $db.roles|select name
         break
         }
     if(!($svr.Logins.Contains($User)))
         {
         Write-Host "$User not a login on $server create it first"
         break
         }
     if (!($db.Users.Contains($User)))
         {
 
         $usr = New-Object ('Microsoft.SqlServer.Management.Smo.User') ($db, $User)
         $usr.Login = $User
         $usr.Create()
 
         $Rol = $db.Roles[$Role]
         $Rol.AddMember($User)
         Write-Host "$User was not a login on $Database on $server"
         Write-Host "$User added to $Database on $Server and $Role Role"
         }
         else
         {
         $Rol = $db.Roles[$Role]
         $Rol.AddMember($User)
         Write-Host "$User added to $Role Role in $Database on $Server "
         }
 }
`

