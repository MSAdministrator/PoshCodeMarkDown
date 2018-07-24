---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 12.0
Encoding: ascii
License: cc0
PoshCode ID: 6532
Published Date: 2016-09-28t01
Archived Date: 2016-10-28t23
---

# import-exceltosql - 

## Description

imports an excel spreadsheet to a sql server table using oledb

## Comments



## Usage



## TODO



## function

`get-exceldata`

## Code

`#
 #
 $filepath = 'C:\Users\u00\Documents\backupset.xlsx'
 $connString = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=`"$filepath`";Extended Properties=`"Excel 12.0 Xml;HDR=YES`";"
 #$connString = "Provider=Microsoft.Jet.OLEDB.4.0;Data Source=`"$filepath`";Extended Properties=`"Excel 8.0;HDR=Yes;IMEX=1`";"
 $qry = 'select * from [backupset$]'
 $sqlserver = 'Z002\SQLEXPRESS'
 $dbname = 'SQLPSX'
 $tblname = 'ExcelData_fill'
  
 #######################
 function Get-ExcelData
 {
  
     param($connString, $qry='select * from [sheet1$]')
  
     $conn = new-object System.Data.OleDb.OleDbConnection($connString)
     $conn.open()
     $cmd = new-object System.Data.OleDb.OleDbCommand($qry,$conn) 
     $da = new-object System.Data.OleDb.OleDbDataAdapter($cmd) 
     $dt = new-object System.Data.dataTable 
     [void]$da.fill($dt)
     $conn.close()
     $dt
  
  
 #######################
 function Write-DataTableToDatabase
 { 
     param($dt,$destServer,$destDb,$destTbl)
 
     $connectionString = "Data Source=$destServer;Integrated Security=true;Initial Catalog=$destdb;"
     $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $connectionString
     $bulkCopy.DestinationTableName = "$destTbl"
     $bulkCopy.WriteToServer($dt)
  
 
 #######################
 $dt = Get-ExcelData $connString $qry
 Write-DataTableToDatabase $dt $sqlserver $dbname $tblname
`

