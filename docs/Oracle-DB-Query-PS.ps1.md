---
Author: ranadip dutta
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6691
Published Date: 2017-01-15t14
Archived Date: 2017-05-25t20
---

# oracle db query ps - 

## Description

oracle database query from powershell

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Download Oracle Data Provider for .NET (ODP.NET). (Just search for �Oracle ODP.NET�.)
 1)Select �Download the latest ODP.NET production release.�
 2)Select �64-bit ODAC Downloads�
 3)Select �ODP.NET_Managed_ODAC12cR4.zip�
 4)Extract the ZIP file to C:\, which creates C:\ODP.NET_Managed_ODAC12cR4.
 5)Run cmd as administrator, navigate to C:\ODP.NET_Managed_ODAC12cR4, and run:
 6)Install_odpm.bat C:\oracle\instantclient_10_2 both
 
 In Powershell, add the DLL and set up a database connection and a query:
 
 Script:@@
 
 Add-Type -Path "C:\Users\User1\ODP.NET_Managed_ODAC12cR4\odp.net\managed\common\Oracle.ManagedDataAccess.dll"
 $username = Read-Host -Prompt "Enter database username"
 $password = Read-Host -Prompt "Enter database password"
 $datasource = Read-Host -Prompt "Enter database TNS name"
 $query = "SELECT first_name, last_name FROM users WHERE last_name = 'Lastname' ORDER BY last_name"
 $connectionString = 'User Id=' + $username + ';Password=' + $password + ';Data Source=' + $datasource
 $connection = New-Object Oracle.ManagedDataAccess.Client.OracleConnection($connectionString)
 $connection.open()
 $command=$connection.CreateCommand()
 $command.CommandText=$query
 $reader=$command.ExecuteReader()
 while ($reader.Read()) {
 $reader.GetString(1) + ', ' + $reader.GetString(0)
 }
 $connection.Close()
 
 
 Expected Output: @@
 
 User1_Firstname, Lastname
 User2_Firstname, Lastname
 User3_Firstname, Lastname
`

