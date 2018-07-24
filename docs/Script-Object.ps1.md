---
Author: justin dearing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: mitl
PoshCode ID: 2587
Published Date: 2012-03-27t16
Archived Date: 2012-02-02t08
---

# script-object.ps1 - 

## Description

a powershell script that will create the create ddl for any object in a sql server database. it requires the open source atlantis.schemaengine.dll available at http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 
 param(
     [Parameter(Mandatory=$true, HelpMessage='The name of the database object we wish to script')]
     [string] $ObjectName,
     [string] $Path = "$($ObjectName).sql",
     [string] $ConnectionString = 'Data Source=.\sqlexpress2k8R2;Initial Catalog=master;Integrated Security=SSPI;',
 );
 
 Add-Type -Path "$($AtlantisSchemaEngineBaseDir)\Atlantis.SchemaEngine\bin\Debug\Atlantis.SchemaEngine.dll"
 $schemaReader = [Atlantis.SchemaEngine.Container.SQLServer.SQLServerSchemaReaderFactory]::GetSpecificSQLServerSchemaReader($ConnectionString, [Atlantis.SchemaEngine.Enumerations.ContainerMode]::Navigation)
 $dbObjects = $schemaReader.ReadObjects() | Where-Object { $_.ObjectName,$_.ObjectDesriptiveName,$_.ObjectQualifiedName -contains $ObjectName };
 if ($dbObjects -eq $null) {
     Throw New-Object System.ArgumentException "Object `"$($ObjectName)`" not found.",'-ObjectName';
 }
 $dbObjects.Script([Atlantis.SchemaEngine.Enumerations.ScriptGenerationType]::Create, (New-Object Atlantis.SchemaEngine.Configuration.GenerationOptions)).Scripts
`

