---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 12.0
Encoding: ascii
License: cc0
PoshCode ID: 1064
Published Date: 2009-04-26t13
Archived Date: 2015-05-18t02
---

# export-splisttosql - 

## Description

exports a sharepoint list to a sql server table using oledb

## Comments



## Usage



## TODO



## function

`get-splist`

## Code

`#
 #
 $connString = 'Provider=Microsoft.ACE.OLEDB.12.0;WSS;IMEX=2;RetrieveIds=Yes; DATABASE=http://sharepoint.acme.com/;LIST={96801432-2d03-42b8-82b0-ac96ca9fea8a};'
 $qry = 'Select Title,Notes,CreateDate, Modified, Created from list'
 
 $sqlserver = 'Z002\SQL2K8'
 $dbname = 'SQLPSX'
 $tblname = 'splist_fill'
 
 #######################
 function Get-SPList
 {
 
     param($connString, $qry='select * from list')
 
     $spConn = new-object System.Data.OleDb.OleDbConnection($connString)
     $spConn.open()
     $cmd = new-object System.Data.OleDb.OleDbCommand($qry,$spConn) 
     $da = new-object System.Data.OleDb.OleDbDataAdapter($cmd) 
     $dt = new-object System.Data.dataTable 
     [void]$da.fill($dt)
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
 
 $dt = Get-SPList $connString $qry
 Write-DataTableToDatabase $dt $sqlserver $dbname $tblname
`

