---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4449
Published Date: 2014-09-09t12
Archived Date: 2014-08-18t21
---

# drop sql users - 

## Description

name

## Comments



## Usage



## TODO



## function

`drop-sqlusers`

## Code

`#
 #
 
 #############################################################################################
 #
 #
 #
 #
 
 
 Function Drop-SQLUsers ($Usr)
 {
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SMO') | out-null
 
 
 $ErrorActionPreference = "SilentlyContinue"
 
 
 $servers = Get-Content 'c:\temp\sqlservers.txt'
 
 
 $logins = @("DOMAIN1\$usr","DOMAIN2\$usr", "DOMAIN3\$usr" , "$usr")
 
 				Write-Host "#################################"
                 Write-Host "Dropping Logins for $Logins" 
 
 
      Write-Host "#########################################"
      Write-Host "`n Database Logins`n"  
 foreach($server in $servers)
 {      	
     $srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
 	foreach($database in $srv.Databases)
 	{
 		if ($database -notlike "*dontwant*")
         {
             foreach($login in $logins)
 		      {
 			 if($database.Users.Contains($login))
 			 {
 			 	$database.Users[$login].Drop();
                  Write-Host " $server , $database , $login  - Database Login has been dropped" 
 			 }
 		      }
 	   }
     }
     }
     
      Write-Host "`n#########################################"
      Write-Host "`n Servers Logins`n" 
       
     foreach($server in $servers)
 {      	
     $srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
 	foreach($login in $logins)
 	{
 		if ($srv.Logins.Contains($login)) 
 		{ 
 			$srv.Logins[$login].Drop(); 
          Write-Host " $server , $login Login has been dropped" 
 		}
 	}
 }
 Write-Host "`n#########################################"
 Write-Host "Dropping Database and Server Logins for $usr - Completed "  
 Write-Host "If there are no logins displayed above then no logins were found or dropped!"    
 Write-Host "###########################################" 
 }
`

