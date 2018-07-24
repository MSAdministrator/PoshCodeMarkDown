---
Author: dfafadfds
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6666
Published Date: 2017-01-04t15
Archived Date: 2017-01-10t16
---

# poweroracle - 

## Description

retrieve data from an oracle database into a dataset.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [System.Reflection.Assembly]::LoadWithPartialName("Oracle.DataAccess")
 
 $ConnectionString = "Data Source=your_server/sid;User Id=user_name;Password=password"
 
 
 $OracleConnection = New-Object Oracle.DataAccess.Client.OracleConnection($ConnectionString)
 $dtSet = New-Object System.Data.DataSet
 $OracleAdapter = New-Object Oracle.DataAccess.Client.OracleDataAdapter($QueryString, $OracleConnection)
 
 [void]$OracleAdapter.Fill($dtSet)
`

