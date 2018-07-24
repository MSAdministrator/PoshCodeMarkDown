---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3337
Published Date: 2013-04-10t11
Archived Date: 2017-04-05t08
---

# sql-update - 

## Description

function uses the microsoft sql cmdlets ‘invoke-sqlcmd’ to connect to a sql database and run an update statement.

## Comments



## Usage



## TODO



## script

`sql-update`

## Code

`#
 #
 <#
 .SYNOPSIS
   Author:...Vidrine
   Date:.....2012/04/08
 .DESCRIPTION
   Function uses the Microsoft SQL cmdlets 'Invoke-SQLcmd' to connect to a SQL database and run an UPDATE statement.
   The target record that will be updated is queried based on a table/column named 'ID'. Simply change this to query based on another value.
 .PARAM server
   Hostname/IP of the server hosting the SQL database.
 .PARAM database
   Name of the SQL database to connect to.
 .PARAM table
   Name of the table within the specified SQL database.
 .PARAM dataField
   Field/Column name from the specified table. This is the field/column that will be updated.
 .PARAM dataValue
   The new value of the field/column that will be updated.
 .PARAM updateID
   The ID of the target record to update.
 .EXAMPLE
 SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 #>
 function SQL-Update {
 param(
 	[string]$server,
 	[string]$database,
 	[string]$table,
 	[string]$dataField,
 	[string]$dataValue,
 	[string]$updateID
 )
 
 $sqlQuery = @"
 UPDATE $database.$table 
 SET $dataField='$dataValue' 
 WHERE id=$updateID
 "@
 
 try {
 Invoke-SQLcmd -ServerInstance $server -Database $database -Query $sqlQuery
 }
 catch {
 }
 }
`

