---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3338
Published Date: 2014-04-10t11
Archived Date: 2014-08-18t21
---

# sql-select - 

## Description

function uses the microsoft sql cmdlets ‘invoke-sqlcmd’ to connect to a sql database and run a select statement.

## Comments



## Usage



## TODO



## script

`sql-select`

## Code

`#
 #
 <#
 .SYNOPSIS
   Author:...Vidrine
   Date:.....2012/04/08
 .DESCRIPTION
   Function uses the Microsoft SQL cmdlets 'Invoke-SQLcmd' to connect to a SQL database and run a SELECT statement.
   Results of the query are returned. Store returned results in a variable to be able to interact with them as an object.
 .PARAM server
   Hostname/IP of the server hosting the SQL database.
 .PARAM database
   Name of the SQL database to connect to.
 .PARAM table
   Name of the table within the specified SQL database.
 .PARAM selectWhat
   [String] Select string. This will specify the SELECT value to use in the SQL statement. Default value is '*'.
 .PARAM where
   [String] Where string. This will specify the WHERE clause to use in the SQL statement. ie. "id='64'"
 .EXAMPLE
 $results = SQL-Select -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -selectWhat '*'
 .EXAMPLE
 $results = SQL-Select -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -selectWhat '*' -where "id='64'"
 #>
 function SQL-Select {
 param(
 	[string]$server,
 	[string]$database,
 	[string]$table,
 	[string]$selectWhat = '*',
 	[string]$where
 	
 )
 
 if ($where){
 $sqlQuery = @"
 SELECT $selectWhat 
 FROM $table 
 WHERE $where
 "@
 }
 
 else {
 $sqlQuery = @"
 SELECT $selectWhat 
 FROM $table
 "@
 }
 
 try
 {
 $results = Invoke-SQLcmd -ServerInstance $server -Database $database -Query $sqlQuery
 
 return $results
 }
 catch{}
 }
`

