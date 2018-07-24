---
Author: dragonmc77
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5962
Published Date: 2016-08-03t19
Archived Date: 2016-05-17t12
---

# execute-sqlcommand - 

## Description

simple function that executes a command (stored procedure) against an sql database.

## Comments



## Usage



## TODO



## function

`execute-sqlcommand`

## Code

`#
 #
 
 	$sqlConnection = New-Object System.Data.SqlClient.SqlConnection
 	$sqlConnection.ConnectionString = "Integrated Security=SSPI;Persist Security Info=False;User ID=ml;Initial Catalog=$Database;Data Source=$Server"
 	
 	$Command.Connection = $sqlConnection
 	
 	$sqlConnection.Open()
 	$Result = $Command.ExecuteNonQuery()
 	$sqlConnection.Close()
 	
 	if ($Result -gt 0) {return $TRUE} else {return $FALSE}
 }
`

