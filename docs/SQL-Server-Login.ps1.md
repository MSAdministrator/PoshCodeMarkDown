---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4431
Published Date: 2014-09-01t09
Archived Date: 2014-08-18t21
---

# sql server login - show-sqluserlogins.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-sqluserlogins`

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 
 
 Function Show-SQLUserLogins ($usr)
 {
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SMO') | out-null
 
 
 $ErrorActionPreference = "SilentlyContinue"
 
 
 $servers = Get-Content 'c:\temp\sqlservers.txt'
 
 
 $logins = @("DOMAIN1\$usr","DOMAIN2\$usr", "DOMAIN3\$usr" , "$usr")
 
 				Write-Host "#################################" 
                 Write-Host "SQL Servers, Databases and Logins for `n$logins displayed below " 
 
        Write-Host " Server Logins`n"
          foreach($server in $servers)
 {
     $srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
     
     		foreach($login in $logins)
 		{
     
     			if($srv.Logins.Contains($login))
 			{
 
                 Write-Host " $server , $login " 
 
 			}
             
             else
             {
 
             }
          }
 }
       Write-Host "`n#########################################"
      Write-Host "`n Database Logins`n"               
 foreach($server in $servers)
 {
 	$srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
     
 	foreach($database in $srv.Databases)
 	{
 		foreach($login in $logins)
 		{
 			if($database.Users.Contains($login))
 			{
 
                 Write-Host " $server , $database , $login " 
 
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

