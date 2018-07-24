---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1446
Published Date: 
Archived Date: 2009-11-30t18
---

# get-procedurecalltree - 

## Description

uses sqlparser.ps1 script http

## Comments



## Usage



## TODO



## script

`invoke-coalesce`

## Code

`#
 #
 
 
 param ($procedure, $server, $database, $schema='dbo')
 
 add-type -AssemblyName Microsoft.SqlServer.Smo
 
 if (!($global:__SQLParser))
 {
     $global:__SQLParser = ./SQLParser.ps1
 }
 
 #######################
 function Invoke-Coalesce
 {
     param ($expression1, $expression2)
 
     if ($expression1)
     { $expression1 }
     else
     { $expression2 }
 
 
 #######################
 filter Get-StatementByType
 {
     param ($statementType)
 
     if ($_)
     { $statement = $_ }
     
     if ($statement | Get-Member -Type Property $statementType)
     { $_.$statementType }
 
     else
     {
         $property = $statement | Get-Member | where {$_.Definition -like "Microsoft.Data.Schema.ScriptDom.Sql.StatementList*"}
         if ($property)
         { $property | foreach {$statement.$($_.Name)} | foreach {$_.Statements} | Get-StatementByType $statementType }
     }
 }
 
 #######################
 function Get-ProcedureReference
 {
     param ($procedure, $procedureText, $server, $database, $schema)
 
     $srv = new-object ("Microsoft.SqlServer.Management.Smo.Server") $server
 
     $sqlparser = switch ($srv.Version.Major)
     {
         8       { new-object SQLParser Sql80,$false,$procedureText  }
         9       { new-object SQLParser Sql90,$false,$procedureText  }
         10      { new-object SQLParser Sql100,$false,$procedureText }
         default { new-object SQLParser Sql100,$false,$procedureText }
     }
 
     $sqlparser.Fragment.Batches | foreach {$_.Statements}  | Get-StatementByType 'ExecutableEntity' | foreach {$_.ProcedureReference.Name}  |
     select @{n='Server';e={Invoke-Coalesce $_.ServerIdentifier.Value $server}}, `
     @{n='Database';e={Invoke-Coalesce $_.DatabaseIdentifier.Value $database}}, `
     @{n='Schema';e={Invoke-Coalesce $_.SchemaIdentifier.Value $schema}}, @{n='Procedure';e={$_.BaseIdentifier.Value}} | 
     select *, @{n='Source';e={"{0}.{1}.{2}.{3}" -f $server,$database,$schema,$procedure}}, `
     @{n='Target';e={"{0}.{1}.{2}.{3}" -f $_.Server,$_.Database,$_.Schema,$_.Procedure}}
 
 
 #######################
 function Get-ProcedureText
 {
     param($server, $database, $schema, $procedure)
     
     $srv = new-object ("Microsoft.SqlServer.Management.Smo.Server") $server
     $db= $srv.Databases[$database]
     $proc = $db.StoredProcedures | where {$_.Schema -eq $schema -and $_.Name -eq $procedure}
     $proc.Script()
  
 
 #######################
 #######################
 $procedureText = Get-ProcedureText $server $database $schema $procedure 
 $procedureReference = Get-ProcedureReference $procedure $procedureText[2] $server $database $schema
 $procedureReference
 if ($procedureReference)
 { $procedureReference | foreach {./Get-ProcedureCallTree.ps1 $_.Procedure $_.Server $_.Database $_.Schema} }
`

