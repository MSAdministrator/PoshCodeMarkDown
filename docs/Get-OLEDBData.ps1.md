---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 12.0
Encoding: ascii
License: cc0
PoshCode ID: 3715
Published Date: 2013-10-26t11
Archived Date: 2016-06-20t19
---

# get-oledbdata - 

## Description

this function is used to retrieve data via an oledb data

## Comments



## Usage



## TODO



## script

`get-oledbdata`

## Code

`#
 #
 ###########################################################################
 #
 #
 #"Provider=Microsoft.ACE.OLEDB.12.0;Data Source=`"$filepath`";Extended Properties=`"Excel 12.0 Xml;HDR=YES`";"
 #"Provider=Microsoft.Jet.OLEDB.4.0;Data Source=`"$filepath`";Extended Properties=`"Excel 8.0;HDR=Yes;IMEX=1`";"
 #'select * from [sheet1$]'
 #"password=$password;User ID=$userName;Data Source=$dbName@$serverName;Persist Security Info=true;Provider=Ifxoledbc.2"
 #"password=$password;User ID=$userName;Data Source=$serverName;Provider=OraOLEDB.Oracle"
 #"Server=$serverName;Trusted_connection=yes;database=$dbname;Provider=SQLNCLI;"
 ###########################################################################
 function Get-OLEDBData ($connectstring, $sql) {
    $OLEDBConn = New-Object System.Data.OleDb.OleDbConnection($connectstring)
    $OLEDBConn.open()
    $readcmd = New-Object system.Data.OleDb.OleDbCommand($sql,$OLEDBConn)
    $readcmd.CommandTimeout = '300'
    $da = New-Object system.Data.OleDb.OleDbDataAdapter($readcmd)
    $dt = New-Object system.Data.datatable
    [void]$da.fill($dt)
    $OLEDBConn.close()
    return $dt
 }
`

