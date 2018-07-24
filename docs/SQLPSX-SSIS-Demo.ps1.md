---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1769
Published Date: 2010-04-12t19
Archived Date: 2016-09-11t16
---

# sqlpsx ssis demo - 

## Description

a powershell script that demonstrates working with ssis using the sql server powershell extensions module ssis.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 add-type -AssemblyName "Microsoft.SqlServer.ManagedDTS, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91"
 
 import-module SSIS
 
 $packages = dir "C:\Program Files\Microsoft SQL Server\100\DTS\Packages\*" | select -ExpandProperty Fullname | foreach {get-ispackage -path $_ }
 $packages | foreach {$package = $_; $_.Configurations | Select @{n='Package';e={$Package.DisplayName}}, Name,ConfigurationString}
 $packages | foreach {$package = $_; $_.Connections | Select @{n='Package';e={$Package.DisplayName}}, Name,ConnectionString}
 
 new-isitem "\msdb" "sqlpsx" "Z002"
 copy-isitemfiletosql -path "C:\Program Files\Microsoft SQL Server\100\DTS\Packages\*" -destination "msdb\sqlpsx" -destinationServer "Z002" -connectionInfo @{SSISCONFIG=".\SQLEXPRESS"}
 
 $packages = get-isitem -path '\sqlpsx' -topLevelFolder 'msdb' -serverName "Z002\SQL2K8" | where {$_.Flags -eq 'Package'} | foreach {get-ispackage -path $_.literalPath -serverName $_.Servername}
 $packages | foreach {$package = $_; $_.Configurations | Select @{n='Package';e={$Package.DisplayName}}, Name,ConfigurationString}
 $packages | foreach {$package = $_; $_.Connections | Select @{n='Package';e={$Package.DisplayName}}, Name,ConnectionString}
`

