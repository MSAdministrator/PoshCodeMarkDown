---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 115
Published Date: 
Archived Date: 2009-11-12t15
---

# start-sql - 

## Description

a series of functions to handle getting data from sql servers.  think of these more as tutorials for using sql server from powershell than as a finished set of scripts.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################################################
 #
 ###################################################################################################
 #
 #
 
 param( 
    $Server = $(Read-Host "SQL Server Name"), 
    $Database = $(Read-Host "Default Database"),  
    $UserName, $Password, $Query )
 
 
 #
 #
 function global:Set-SqlConnection( $Server = $(Read-Host "SQL Server Name"), $Database = $(Read-Host "Default Database"),  $UserName , $Password  ) {
 
   if( ($UserName -gt $null) -and ($Password -gt $null)) {
     $login = "User Id = $UserName; Password = $Password"
   } else {
     $login = "Integrated Security = True"
   }
   $SqlConnection.ConnectionString = "Server = $Server; Database = $Database; $login"
 }
 
 #
 #
 function global:Get-SqlDataTable( $Query = $(Read-Host "Enter SQL Query")) {
   if (-not ($SqlConnection.State -like "Open")) { $SqlConnection.Open() }
   $SqlCmd = New-Object System.Data.SqlClient.SqlCommand $Query, $SqlConnection
 
   $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
   $SqlAdapter.SelectCommand = $SqlCmd
 
   $DataSet = New-Object System.Data.DataSet
   $SqlAdapter.Fill($DataSet) | Out-Null
 
   $SqlConnection.Close()
   
   return $DataSet.Tables[0]
 }
 
 #
 #
 function global:Get-SQLQuery($Query = $(Read-Host "Enter SQL Query"), $type = "string")
 {
   if (-not ($SqlConnection.State -like "Open")) { $SqlConnection.Open() }
   $SqlCmd = New-Object System.Data.SqlClient.SqlCommand $Query, $SqlConnection
 
 
   
   $dr = $SqlCmd.ExecuteReader()
   while($dr.Read())
   {
     #$results += $dr.GetValue(0) -as $type
   }
   $dr.Close()
   $dr.Dispose()
 
 }
 
 Set-Variable SqlConnection (New-Object System.Data.SqlClient.SqlConnection) -Scope Global -Option AllScope -Description "Personal variable for Sql Query functions"
 
 Set-SqlConnection $Server $Database
 
 if( $query -gt $null ) {
   Get-SqlDataTable $Query
 }
 
 Set-Alias gdt Get-SqlDataTable  -Option AllScope -scope Global -Description "Personal Function alias from Get-Sql.ps1"
 Set-Alias sql Set-SqlConnection -Option AllScope -scope Global -Description "Personal Function alias from Get-Sql.ps1"
 Set-Alias gq  Get-SqlQuery      -Option AllScope -scope Global -Description "Personal Function alias from Get-Sql.ps1"
`

