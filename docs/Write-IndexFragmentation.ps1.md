---
Author: brandon warner
Publisher: 
Copyright: 
Email: 
Version: 2016.03.02
Encoding: ascii
License: cc0
PoshCode ID: 6424
Published Date: 2016-06-28t15
Archived Date: 2016-07-01t07
---

# write-indexfragmentation - 

## Description

collects information about index fragmentation and writes the results to a csv file

## Comments



## Usage



## TODO



## script

`write-indexfragmentationreport`

## Code

`#
 #
 <#
 .SYNOPSIS
 Analyzes SQL Server Index Fragmentation and creates a csv report
 
 .DESCRIPTION
 Collects Information about index fragmentation and writes the results to a csv file 
 
 .PARAMETER ServerInstance
 The Server\Instance which you want to analyze the index fragmentation on.
 
 .PARAMETER DestFolderPath
 The destination folder path
 
 .EXAMPLE
 Write-IndexFragmentationReport -ServerInstance 'MyServerInstance' -DestFolderPath 'C:\TEMP' -Verbose
 
 .NOTES
 +---------------------------------------------------------------------------------------------+
 | REVISION HISTORY:                                                                           |
 +---------------------------------------------------------------------------------------------+
 |   DATE       AUTHOR          CHANGE DESCRIPTION                                             |
 |   ---------- --------------- -------------------------------------------------------------- |
     2016.03.02 Brandon Warner  Initial Draft      
 +---------------------------------------------------------------------------------------------+
 | UNIT TESTING SCRIPTS:                                                                       |
    
 +---------------------------------------------------------------------------------------------+
 #>
 
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.Smo') | Out-Null
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SmoExtended') | Out-Null
 
 function Write-IndexFragmentationReport
   {
 
     [CmdletBinding()]
     Param
       (
           [Parameter(Mandatory = $true, Position = 1)]
           [string]$ServerInstance
 
         , [Parameter(Mandatory = $true, Position = 2)]
           [string]$DestFolderPath
       )
 
     $s = New-Object Microsoft.SqlServer.Management.Smo.Server $ServerInstance
     $s.ConnectionContext.Disconnect() | Out-Null
     $s.ConnectionContext.ApplicationName     = 'PowerShell Script'
     $s.ConnectionContext.LoginSecure         = $true
     $s.ConnectionContext.ConnectTimeout = (60*10)
     $s.ConnectionContext.Connect()
   
     $results = @()
 
     $s.Databases | where {$_.IsSystemObject -ne $true} | %{
     
       $DatabaseID = $_.ID
       $DatabaseName = $_.Name
       $_.Tables | where {$_.IsSystemObject -ne $true} | 
         %{ 
           $tableName = $_.Name
           $ObjectID = $_.ID
           "Getting Index Data for $DatabaseName.$tableName" | oh
           $_.Indexes | 
             %{
                 $IndexID = $_.ID
                 "Analyzing index $($_.Name)" | oh
                 $thisIndex = $_
                 $frag_query = @"
 SELECT
     avg_fragmentation_in_percent
   , page_count
   , avg_page_space_used_in_percent
 FROM
   sys.dm_db_index_physical_stats($($DatabaseID), $($ObjectID), $($IndexID), NULL, 'LIMITED')
 "@
         $frag_query | Write-Verbose
         $query_results                    = $s.Databases[$DatabaseName].ExecuteWithResults($frag_query)
         $page_count                       = $query_results.Tables[0].Rows[0].page_count
         $avg_fragmentation_in_percent     = $query_results.Tables[0].Rows[0].avg_fragmentation_in_percent
         $avg_page_space_used_in_percent   = $query_results.Tables[0].Rows[0].avg_page_space_used_in_percent
         
         $results +=  New-Object -TypeName PSObject -Property @{
             DatabaseName              = $DatabaseName
             TableName                 = $tableName
             IndexName                 = $thisIndex.Name
             IndexType                 = $_.IndexType                  
             AvgFragmentationInPercent = $avg_fragmentation_in_percent    
             AvgPageSpaceUsedPercent   = $avg_page_space_used_in_percent
             SizeMB                    = [Math]::Truncate(($page_count*8)/1024)
             FillFactor                = $thisIndex.FillFactor
           }
         
         }
       }
     }
     if($DestFolderPath.Substring($DestFolderPath.Length - 1, 1) -eq '\')
       { $DestFolderPath = $DestFolderPath.Substring(0,$DestFolderPath.Length-1) }
     $results | select DatabaseName,TableName,IndexName,IndexType,SizeMB,AvgFragmentationInPercent,AvgPageSpaceUsedPercent | 
       Export-Csv "$DestFolderPath\$($ServerInstance -replace '\\','_')_FragmentationReport_$((Get-Date).ToString("yyyy.MM.dd.HH.mm.ss")).csv" -Force -NoTypeInformation
   }
`

