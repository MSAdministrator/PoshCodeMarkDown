---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1139
Published Date: 
Archived Date: 2009-06-07t23
---

# librarysqldata - 

## Description

library defines two simple functions that use ado.net to get and set sql data.

## Comments



## Usage



## TODO



## function

`get-sqldata`

## Code

`#
 #
 #######################
 function Get-SqlData
 {
     param([string]$serverName=$(throw 'serverName is required.'), [string]$databaseName=$(throw 'databaseName is required.'),
           [string]$query=$(throw 'query is required.'))
 
     Write-Verbose "Get-SqlData serverName:$serverName databaseName:$databaseName query:$query"
 
     $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;"
     $da = New-Object "System.Data.SqlClient.SqlDataAdapter" ($query,$connString)
     $dt = New-Object "System.Data.DataTable"
     [void]$da.fill($dt)
     $dt
 
 
 #######################
 function Set-SqlData
 {
 
      param([string]$serverName=$(throw 'serverName is required.'), [string]$databaseName=$(throw 'databaseName is required.'),
           [string]$query=$(throw 'query is required.'))
 
     $connString = "Server=$serverName;Database=$databaseName;Integrated Security=SSPI;"
     $conn = new-object System.Data.SqlClient.SqlConnection $connString
     $conn.Open()
     $cmd = new-object System.Data.SqlClient.SqlCommand("$query", $conn)
     [void]$cmd.ExecuteNonQuery()
     $conn.Close()
 
`

