---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3006
Published Date: 2012-10-17t12
Archived Date: 2016-05-21t13
---

# write-fileinfotosql.ps1 - 

## Description

demonstration script for getting file names and last update times into a sql server table.

## Comments



## Usage



## TODO



## script

`clear-fill`

## Code

`#
 #
 
 $sqlserver = 'SQL1' 
 $dbname = 'DataAdmin'
 $tblname = 'data_model_fill'
 
 #######################
 function Clear-Fill
 {
 
     $connString = "Server=$sqlserver;Database=$dbname;Integrated Security=SSPI;"
     $conn = new-object System.Data.SqlClient.SqlConnection $connString
     $conn.Open()
     $cmd = new-object System.Data.SqlClient.SqlCommand("TRUNCATE TABLE $tblname", $conn)
     $cmd.ExecuteNonQuery()
     $conn.Close()
 
 
 #######################
 function Out-DataTable 
 {
     param($Properties="*")
     Begin
     {
         $dt = new-object Data.datatable  
         $First = $true 
     }
     Process
     {
         $DR = $DT.NewRow()  
         foreach ($item in $_ |  Get-Member -type *Property $Properties ) {  
           $name = $item.Name
           if ($first) {  
             $Col =  new-object Data.DataColumn  
             $Col.ColumnName = $name
             $DT.Columns.Add($Col)
           }  
             $DR.Item($name) = $_.$name  
         }  
         $DT.Rows.Add($DR)  
         $First = $false  
     }
     End
     {
         return @(,($dt))
     }
 
 
 #######################
 function Write-DataTableToDatabase
 { 
     param($destServer,$destDb,$destTbl)
     process
     {
         $connectionString = "Data Source=$destServer;Integrated Security=true;Initial Catalog=$destdb;"
         $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $connectionString
         $bulkCopy.DestinationTableName = "$destTbl"
         $bulkCopy.WriteToServer($_)
     }
 
 
 #######################
 Clear-Fill
 get-childitem "\\z001.contoso.com\wwwroot`$\Playbook\Databases\DataModels" | where {$_.PSIsContainer -eq $true} | select name, @{name='htm';Expression={test-path "\\z001.contoso.com\wwwroot`$\Playbook\Databases\DataModels\$($_.Name)\$($_.Name).htm"}}, @{name='lastWrite';Expression={if (test-path "\\z001.contoso.com\wwwroot`$\Playbook\Databases\DataModels\$($_.Name)\$($_.Name).htm") {$last = Get-Item "\\z001.contoso.com\wwwroot`$\Playbook\Databases\DataModels\$($_.Name)\$($_.Name).htm" | Select LastWriteTime;$last.LastWriteTime}}} |  
 Out-DataTable | Write-DataTableToDatabase $sqlserver $dbname $tblname
`

