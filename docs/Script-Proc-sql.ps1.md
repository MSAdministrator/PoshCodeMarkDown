---
Author: justin dearing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: mitl
PoshCode ID: 2582
Published Date: 2011-03-26t12
Archived Date: 2014-08-18t21
---

# script-proc.sql - 

## Description

a script that scripts a stored proc in sql server.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 
 
 param(
     [Parameter(Mandatory=$true,HelpMessage='The name of the stored procedure or user defined function we wish to script')]
     [string] $ProcedureName,
     [string] $Path = "$($ProcedureName).sql",
     [string] $ConnectionString = 'Data Source=.\sqlexpress2k8R2;Initial Catalog=master;Integrated Security=SSPI;'
 );
 
 try {
     [System.Data.SqlClient.SqlConnection] $cn = New-Object System.Data.SqlClient.SqlConnection (,$ConnectionString);
     $cn.Open() > $null;
     $cmd = $cn.CreateCommand();
     $cmd.CommandType = [System.Data.CommandType]::StoredProcedure;
     $cmd.CommandText = 'sp_helptext';
     $cmd.Parameters.AddWithValue('@objname', $ProcedureName) > $null;
     [System.Data.IDataReader] $rdr = $cmd.ExecuteReader();
     [string] $sproc_text = '';
     while ($rdr.Read()) {
         $sproc_text += $rdr[0];
     }
     $rdr.Close();
     $sproc_text | Out-File -FilePath $Path;
 }
 finally {
     if ($cmd -ne $null) { $cmd.Dispose(); }
     if ($cn -ne $null) { $cn.Dispose(); }
 }
`

