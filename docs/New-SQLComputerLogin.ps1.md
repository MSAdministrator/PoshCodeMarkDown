---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3012
Published Date: 2011-10-18t11
Archived Date: 2011-10-23t11
---

# new-sqlcomputerlogin - 

## Description

create a computer login on a sql server, optionally forcing the issue by removing any pre-existing account.  for some reason, we run into this all the time with renamed computers…

## Comments



## Usage



## TODO



## function

`new-sqlcomputerlogin`

## Code

`#
 #
 
 function New-SqlComputerLogin {
 #.Synopsis
 #.Example
 #
 [CmdletBinding()]
 param(
 	[Parameter(Mandatory=$true, Position=0)]
 	[String]$SQLServer,
 	
 	[Parameter(ValueFromPipelineByPropertyName=$true, Position=1)]
 	[String]$ComputerName,
 	
 	[Switch]$Force
 )
 process {
 $Computer = Get-ADComputer $ComputerName
 #$NTAccountName = $Computer.NTAccountName
 
 
 invoke-sqlcmd2 -ServerInstance $SQLServer -Query "CREATE LOGIN [$($Computer.NTAccountName)] FROM WINDOWS" -ErrorVariable SqlError -ErrorAction SilentlyContinue
 if($SqlError[0].Exception.GetBaseException().Message.EndsWith("already exists.")) {
 	if(!$Force) {
 		Write-Error $SqlError[0].Exception.GetBaseException().Message
 		$ExistingAccount = 
 			invoke-sqlcmd2 -ServerInstance $SQLServer -Query "select name, sid from sys.server_principals where type IN ('U','G')" | 
 				add-member -membertype ScriptProperty SSDL { new-object security.principal.securityidentifier $this.SID, 0 } -passthru | 
 				where-object {$_.SSDL -eq $Computer.SID}
 
 		Write-Warning "You need to drop [$($ExistingAccount.Name)] or run New-SqlComputerLogin again with -Force"
 	} else {
 		Write-Warning ($SqlError[0].Exception.GetBaseException().Message + " -- DROPping and reCREATEing")
 		$ExistingAccount = 
 			invoke-sqlcmd2 -ServerInstance $SQLServer -Query "select name, sid from sys.server_principals where type IN ('U','G')" | 
 				add-member -membertype ScriptProperty SSDL { new-object security.principal.securityidentifier $this.SID, 0 } -passthru | 
 				where-object {$_.SSDL -eq $Computer.SID}
 
 		Write-Warning "Dropping [$($ExistingAccount.Name)] to create [$($Computer.NTAccountName)]"
 		invoke-sqlcmd2 -ServerInstance $SQLServer -Query "DROP LOGIN [$($ExistingAccount.Name)]" -ErrorAction Stop
 
 		invoke-sqlcmd2 -ServerInstance $SQLServer -Query "CREATE LOGIN [$($Computer.NTAccountName)] FROM WINDOWS"
 		invoke-sqlcmd2 -ServerInstance $SQLServer -Query "select name, type_desc, default_database_name, create_date from sys.server_principals where name = '$($Computer.NTAccountName)'" 
 	}
 }
 }
 }
`

