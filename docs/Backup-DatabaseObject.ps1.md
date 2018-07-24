---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3367
Published Date: 2013-04-17t15
Archived Date: 2016-01-28t13
---

# backup-databaseobject - 

## Description

the backup-databaseobject function backs up a database object definition by scripting out the object to a .sql text file.

## Comments



## Usage



## TODO



## script

`backup-databaseobject`

## Code

`#
 #
 add-type -AssemblyName "Microsoft.SqlServer.ConnectionInfo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
 add-type -AssemblyName "Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
 add-type -AssemblyName "Microsoft.SqlServer.SMOExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
 add-type -AssemblyName "Microsoft.SqlServer.SqlEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
 add-type -AssemblyName "Microsoft.SqlServer.Management.Sdk.Sfc, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
 
 
 #######################
 <#
 .SYNOPSIS
 Backs up a database object definition.
 .DESCRIPTION
 The Backup-DatabaseObject function backs up a database object definition by scripting out the object to a .sql text file.
 .EXAMPLE
 Backup-DatabaseObject -ServerInstance Z002 -Database AdventureWorks -Schema HumanResources -Name vEmployee -Path "C:\Users\Public"
 This command backups up the vEmployee view to a .sql file.
 .NOTES 
 Version History 
 v1.0   - Chad Miller - Initial release 
 #>
 function Backup-DatabaseObject
 {
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$true)]
     [ValidateNotNullorEmpty()]
     [string]$ServerInstance,
     [Parameter(Mandatory=$true)]
     [ValidateNotNullorEmpty()]
     [string]$Database,
     [Parameter(Mandatory=$true)]
     [ValidateNotNullorEmpty()]
     [string]$Schema,
     [Parameter(Mandatory=$true)]
     [ValidateNotNullorEmpty()]
     [string]$Name,
     [Parameter(Mandatory=$true)]
     [ValidateNotNullorEmpty()]
     [string]$Path
     )
     
     $server = new-object Microsoft.SqlServer.Management.Smo.Server($ServerInstance)
     $db = $server.Databases[$Database]
 
     $urns = new-object Microsoft.SqlServer.Management.Smo.UrnCollection
 
     $db.enumobjects() | where {$_.schema -eq $Schema -and  $_.name -eq $Name } |
         foreach {$urn = new-object Microsoft.SqlServer.Management.Sdk.Sfc.Urn($_.Urn);
                  $urns.Add($urn) }
 
     if ($urns.Count -gt 0) {
         
         $scripter = new-object Microsoft.SqlServer.Management.Smo.Scripter($server)
         
         $scripter.options.ScriptBatchTerminator = $true
         $scripter.options.FileName = "$Path\BEFORE_$Schema.$Name.sql"
         $scripter.options.ToFileOnly = $true
         $scripter.options.Permissions = $true
         $scripter.options.DriAll = $true
         $scripter.options.Triggers = $true
         $scripter.options.Indexes = $true
         $scripter.Options.IncludeHeaders = $true
         
         $scripter.Script($urns)
         
     }
     else {
         write-warning "Object $Schema.$Name Not Found!"
     }
 
`

