---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4433
Published Date: 2013-09-01t10
Archived Date: 2016-10-09t19
---

# sql user permissions - show-sqluserpermissions.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-sqluserpermissions`

## Code

`#
 #
 
 #############################################################################################
 #
 #
 
 
 Function Show-SQLUserPermissions ($usr)
 {
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SMO') | out-null
 
 
 $ErrorActionPreference = "SilentlyContinue"
 
 
 $servers = Get-Content 'c:\temp\sqlservers.txt'
 
 
 $logins = @("DOMAIN1\$usr","DOMAIN2\$usr", "DOMAIN3\$usr" , "$usr")
 
 				Write-Host "#################################" 
                 Write-Host "Logins for `n $logins displayed below" 
 
        Write-Host " Server Logins"
          foreach($server in $servers)
 {
     $srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
     
     		foreach($login in $logins)
 		{
     
     			if($srv.Logins.Contains($login))
 			{
 
                                     Write-Host "`n $server , $login " 
                                             foreach ($Role in $Srv.Roles)
                                 {
                                     $RoleMembers = $Role.EnumServerRoleMembers()
                                     
                                         if($RoleMembers -contains $login)
                                         {
                                         Write-Host " $login is a member of $Role on $Server"
                                         }
                                 }
 
 			}
             
             else
             {
 
             }
          }
 }
       Write-Host "`n#########################################"
      Write-Host "`n Database Logins"               
 foreach($server in $servers)
 {
 	$srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
     
 	foreach($database in $srv.Databases)
 	{
 		foreach($login in $logins)
 		{
 			if($database.Users.Contains($login))
 			{
                                        Write-Host "`n $server , $database , $login " 
                         foreach($role in $Database.Roles)
                                 {
                                     $RoleMembers = $Role.EnumMembers()
                                     
                                         if($RoleMembers -contains $login)
                                         {
                                         Write-Host " $login is a member of $Role Role on $Database on $Server"
                                         }
                                 }
                     
 
 			}
                 else
                     {
                         continue
                     }   
            
 		}
 	}
     }
    Write-Host "`n#########################################"
    Write-Host "Finished - If there are no logins displayed above then no logins were found!"    
    Write-Host "#########################################" 
 
 
 
 
 
 }
`

