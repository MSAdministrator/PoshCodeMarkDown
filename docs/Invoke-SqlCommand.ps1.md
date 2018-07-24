---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 8.0
Encoding: ascii
License: cc0
PoshCode ID: 2188
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# invoke-sqlcommand.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Return the results of a SQL query or operation
 
 .EXAMPLE
 
 Invoke-SqlCommand.ps1 -Sql "SELECT TOP 10 * FROM Orders"
 Invokes a command using Windows authentication
 
 .EXAMPLE
 
 PS >$cred = Get-Credential
 PS >Invoke-SqlCommand.ps1 -Sql "SELECT TOP 10 * FROM Orders" -Cred $cred
 Invokes a command using SQL Authentication
 
 .EXAMPLE
 
 PS >$server = "MYSERVER"
 PS >$database = "Master"
 PS >$sql = "UPDATE Orders SET EmployeeID = 6 WHERE OrderID = 10248"
 PS >Invoke-SqlCommand $server $database $sql
 Invokes a command that performs an update
 
 .EXAMPLE
 
 PS >$sql = "EXEC SalesByCategory 'Beverages'"
 PS >Invoke-SqlCommand -Sql $sql
 Invokes a stored procedure
 
 .EXAMPLE
 
 Invoke-SqlCommand (Resolve-Path access_test.mdb) -Sql "SELECT * FROM Users"
 Access an Access database
 
 .EXAMPLE
 
 Invoke-SqlCommand (Resolve-Path xls_test.xls) -Sql 'SELECT * FROM [Sheet1$]'
 Access an Excel file
 
 #>
 
 param(
     [string] $DataSource = ".\SQLEXPRESS",
 
     [string] $Database = "Northwind",
 
     [Parameter(Mandatory = $true)]
     [string[]] $SqlCommand,
 
     [int] $Timeout = 60,
 
     $Credential
 )
 
 
 Set-StrictMode -Version Latest
 
 $authentication = "Integrated Security=SSPI;"
 
 if($credential)
 {
     $credential = Get-Credential $credential
     $plainCred = $credential.GetNetworkCredential()
     $authentication =
         ("uid={0};pwd={1};" -f $plainCred.Username,$plainCred.Password)
 }
 
 $connectionString = "Provider=sqloledb; " +
                     "Data Source=$dataSource; " +
                     "Initial Catalog=$database; " +
                     "$authentication; "
 
 if($dataSource -match '\.xls$|\.mdb$')
 {
     $connectionString = "Provider=Microsoft.Jet.OLEDB.4.0; " +
         "Data Source=$dataSource; "
 
     if($dataSource -match '\.xls$')
     {
         $connectionString += 'Extended Properties="Excel 8.0;"; '
 
         if($sqlCommand -notmatch '\[.+\$\]')
         {
             $error = 'Sheet names should be surrounded by square brackets, ' +
                 'and have a dollar sign at the end: [Sheet1$]'
             Write-Error $error
             return
         }
     }
 }
 
 $connection = New-Object System.Data.OleDb.OleDbConnection $connectionString
 $connection.Open()
 
 foreach($commandString in $sqlCommand)
 {
     $command = New-Object Data.OleDb.OleDbCommand $commandString,$connection
     $command.CommandTimeout = $timeout
 
     $adapter = New-Object System.Data.OleDb.OleDbDataAdapter $command
     $dataset = New-Object System.Data.DataSet
     [void] $adapter.Fill($dataSet)
 
     $dataSet.Tables | Select-Object -Expand Rows
 }
 
 $connection.Close()
`

