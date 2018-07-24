---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1062
Published Date: 
Archived Date: 2009-04-29t02
---

# test ora proc wrapper 1 - 

## Description

powershell wrapper to an oracle stored procedure returning one ref cursor. here it determines the version of the oracle server. you can substitute similar procedures.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 
 
 
 [System.Reflection.Assembly]::LoadWithPartialName("Oracle.DataAccess")
 [System.Reflection.Assembly]::LoadWithPartialName("System.Data.OracleClient")
 
 function noparm_recordset ()
 {
 
 $connection = New-Object System.Data.OracleClient.OracleConnection($con_ora)
 $command = New-Object System.Data.OracleClient.OracleCommand('noparm_recordset', $connection)
 $command.CommandType = [System.Data.CommandType]::StoredProcedure
 
 
 #$command.Parameters.Add("ReturnValue", [Data.SqlDBType]::int)  | out-null 
 #$command.Parameters["ReturnValue"].Direction = [System.Data.ParameterDirection]::ReturnValue 
 
 #$command.Parameters.Add("RC_1", [System.Data.OracleClient.OracleType]::Cursor)  | out-null 
 $command.Parameters.Add("RC_1", [Data.OracleClient.OracleType]::Cursor)  | out-null 
 $command.Parameters["RC_1"].Direction = [System.Data.ParameterDirection]::Output 
 
 $connection.Open()  | out-null
 $adapter = New-Object System.Data.OracleClient.OracleDataAdapter $command
 $dataset = New-Object System.Data.DataSet
 [void] $adapter.Fill($dataSet)
 #$command.ExecuteNonQuery()
 $connection.Close() | out-null
 
 $CommandOutput = New-Object PSObject
 $CommandOutput | Add-Member -Name Tables -Value $dataSet.Tables -MemberType NoteProperty
 
 
 #$CommandOutput | Add-Member -Name ReturnValue -Value $command.Parameters["ReturnValue"].Value -MemberType NoteProperty
 
 
 return $CommandOutput 
 
 }
 
 (noparm_recordset).tables[0]
 
 <#
 -- code of the testprocedure 
 create or replace procedure noparm_recordset (
     rc_1 in out SYS_REFCURSOR
 ) as
 begin
     Open rc_1 for
 	SELECT * FROM product_component_version ;
 end;
 /
 
 -- Test call using SQL*Plus
 var r refcursor
 exec noparm_recordset(:r)
 print r
 
 #>
`

