---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 2.3.1
Encoding: ascii
License: cc0
PoshCode ID: 2530
Published Date: 
Archived Date: 2017-05-11t14
---

#  - 

## Description

script demonstrates automated sql database restore using sqlpsx sqlserver module that relocates files to default directory and kills all connections to database.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 param($sqlserver, $filepath)
 
 
 import-module sqlserver -force
 
 
 $server = get-sqlserver $sqlserver
 
 $filepath = Resolve-Path $filepath | select -ExpandProperty Path
 $dbname = Get-ChildItem $filePath | select -ExpandProperty basename
 
 $dataPath = Get-SqlDefaultDir -sqlserver $server -dirtype Data
 $logPath = Get-SqlDefaultDir -sqlserver $server -dirtype Log
 
 $relocateFiles = @{}
 Invoke-SqlRestore -sqlserver $server  -filepath $filepath -fileListOnly | foreach { `
     if ($_.Type -eq 'L')
     { $physicalName = "$logPath\{0}" -f [system.io.path]::GetFileName("$($_.PhysicalName)") }
     else
     { $physicalName = "$dataPath\{0}" -f [system.io.path]::GetFileName("$($_.PhysicalName)") }
     $relocateFiles.Add("$($_.LogicalName)", "$physicalName")
 }
 
 $server.KillAllProcesses($dbname)
 
 Invoke-SqlRestore -sqlserver $server -dbname $dbname -filepath $filepath -relocatefiles $relocateFiles -Verbose -force
`

