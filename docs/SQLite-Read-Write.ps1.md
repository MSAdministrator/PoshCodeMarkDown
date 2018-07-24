---
Author: foureight84
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2879
Published Date: 2011-07-31t01
Archived Date: 2016-10-30t00
---

# sqlite read / write - 

## Description

a very simple example of reading and writing from and to a sqlite db using powershell.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $scriptDir = "Path to your SQLite DLL"
 
 Add-Type -Path "$scriptDir\System.Data.SQLite.dll"
 
 
 ###
 ###
 ###
 ###
 ################################################
 
 ################################################
 ###
 
 
 $SQLite = New-Module { 
 
 	Function querySQLite {
 		param([string]$query)
 
 			$datatSet = New-Object System.Data.DataSet
 			
 			$database = "$scriptDir\data"
 			
 			$connStr = "Data Source = $database"
 			$conn = New-Object System.Data.SQLite.SQLiteConnection($connStr)
 			$conn.Open()
 
 			$dataAdapter = New-Object System.Data.SQLite.SQLiteDataAdapter($query,$conn)
 			[void]$dataAdapter.Fill($datatSet)
 			
 			$conn.close()
 			return $datatSet.Tables[0].Rows
 			
 	}
 	
 	Function writeSQLite {
 		param([string]$query)
 		
 			$database = "$scriptDir\data"
 			$connStr = "Data Source = $database"
 			$conn = New-Object System.Data.SQLite.SQLiteConnection($connStr)
 			$conn.Open()
 			
 			$command = $conn.CreateCommand()
 			$command.CommandText = $query
 			$RowsInserted = $command.ExecuteNonQuery()
 			$command.Dispose()
 	}
 	
 	Export-ModuleMember -Variable * -Function * 
 } -asCustomObject
`

