---
Author: ziembor
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3180
Published Date: 2012-01-21t03
Archived Date: 2012-01-23t05
---

# set-fsrmquotabasedonsql - 

## Description

set-fsrmquotabasedonsql

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $error.clear() 
 cls 
 $dataSource = "Serwer\SQL01"
 $database = "nazwaBazyDanych"
 $authentication = "User Id=sqlLogin;Password=P@sswd1;"
 $connectionString = "Provider=sqloledb; Data Source=$dataSource; Initial Catalog=$database; $authentication;"
 $connection = New-Object System.Data.OleDb.OleDbConnection $connectionString
 $connection.Open()
 $qry = "SELECT ItemName,ServerName,QuotaValue FROM Packages WHERE (SI.ItemTypeID = 2)"
 $commandPrev = New-Object System.Data.OleDb.OleDbCommand $qry,$connection
 $adapterPrev = New-Object System.Data.OleDb.OleDbDataAdapter $commandPrev
 $datasetPrev = New-Object System.Data.DataSet
 [void] $adapterPrev.Fill($dataSetPrev)
 $arr = $dataSetPrev.Tables[0].Rows
 foreach ($row in $arr) {
 $ItemName   = $row.ItemName 
 $ServerName = $row.ServerName 
 $QuotaValue = $row.QuotaValue.ToString()
 $QuotaValueMB = $QuotaValue + "MB"
 dirquota quota add  /Path:$ItemName /Limit:$QuotaValueMB  /Remote:$ServerName  /Overwrite /Type:Soft
 dirquota quota list /Path:$ItemName /Remote:$ServerName
 }
 $connection.Close()
 #/3#
`

