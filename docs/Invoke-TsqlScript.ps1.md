---
Author: brandon warner
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6425
Published Date: 2016-06-28t15
Archived Date: 2016-07-01t08
---

# invoke-tsqlscript - 

## Description

runs a t-sql script using sql server management objects (smo). provides various flags to funnel result sets and display messages to different outputs, such as csv file, host, and out-gridview form, etc. displays detailed error messages similar to sql server management studio

## Comments



## Usage



## TODO



## script

`invoke-tsqlscript`

## Code

`#
 #
 <#
 .SYNOPSIS
 Runs a T-SQL Script using SQL Server Management Objects (SMO)
 
 .DESCRIPTION
 Provides various flags to funnel result sets and display messages to different outputs 
 (See Parameters)
 
 .NOTES
 ===============================================================================================
 | ORIGIN STORY                                                                                |
 ===============================================================================================
 |   DATE        : 2016-06-15
 |   AUTHOR      : Brandon W. Warner
 |   DESCRIPTION : Initial Draft
 ===============================================================================================
 
 .PARAMETER ServerInstance
 The target Server\Instance that we want to run the script on
 
 .PARAMETER Database
 The target database name that we want to run the script on
 
 .PARAMETER Path
 The path of the script that we want to run
 
 .PARAMETER WriteResultSetsToHost
 Switch that causes the result sets to be piped as table formatted text to the host
 
 .PARAMETER DisplayResultsToOGV
 Displays each result set of the script to an Out-GridView dialog window
 
 .PARAMETER LogInfoMessagesToFile
 Switch that logs the info message output to a file in addition to returning these messages
 to the host
 
 .PARAMETER ExportResultsAsCsv
 Switch that exports the result sets of the script execution to a set of .csv files
 
 .EXAMPLE 
 Invoke-TsqlScript `
   -Path "$env:USERPROFILE\MyScript.sql" `
   -ServerInstance 'MyServer\Instance' `
   -Database 'MyDatabase' `
   -DisplayResultsToOGV | Out-Null
 
 .EXAMPLE 
 Invoke-TsqlScript `
   -TSQL 'SELECT TOP 100 * FROM dbo.MyTable' `
   -ServerInstance 'MyServer\Instance' `
   -Database 'MyDatabase' `
   -DisplayResultsToOGV | Out-Null
 
 #>
 
 function Invoke-TsqlScript
   {
     [OutputType([System.Data.DataSet])]
     [CmdletBinding( DefaultParameterSetName='ByScriptFilePath')]
     Param
       (
           [Parameter(Mandatory = $true)]
           [string]$ServerInstance
   
         , [Parameter(Mandatory = $true)]
           [string]$Database
   
         , [Parameter(Mandatory = $true, ParameterSetName = 'ByScriptFilePath')]          
           [string]$Path
 
         , [Parameter(Mandatory = $true, ParameterSetName = 'ByTSQLString')]
           [string]$TSQL  
   
         , [Parameter(Mandatory = $false)]
           [Switch]$DisplayResultsToOGV
 
         , [Parameter(Mandatory = $false)]
           [Switch]$DisplayResultsToHost
   
         , [Parameter(Mandatory = $false)]
           [Switch]$LogInfoMessagesToFile
   
         , [Parameter(Mandatory = $false)]
           [Switch]$ExportResultsAsCsv  
   
         , [Parameter(Mandatory = $false)]
           [Switch]$SingleLineMessageOutputMode  
       )
 
     [void][System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SMO')
     
     switch($PSCmdlet.ParameterSetName)
       {
         'ByScriptFilePath'  { $script_contents = Get-Content -Path $Path -Raw}
         'ByTSQLString'      { $script_contents = $TSQL }
       }
   
     $sql_exec_header = @"
 ####################################################################################################
 "@
     Write-Host $sql_exec_header 
     if($LogInfoMessagesToFile)
       {
         $sql_exec_header | Out-File "$Path.OUT.txt" -Append
       }  
 
     $tsql_to_run = $script_contents -split '\bGO\b' 
 
     $Batch = New-Object -TypeName:Collections.Specialized.StringCollection
     $Batch.AddRange($tsql_to_run)
 
     $SmoServer = New-Object Microsoft.SqlServer.Management.Smo.Server $ServerInstance;
     if(!$SmoServer)
         {
           Write-Error -Message "Unable to reach Server\Instance: $ServerInstance"
         }
     $SmoServer.ConnectionContext.SqlConnectionObject.FireInfoMessageEventOnUserErrors = $true      
     $SqlClientMessageHandler = 
       [System.Data.SqlClient.SqlInfoMessageEventHandler]{
         param
           (
               $sender
             , $event
           ) 
         $event.Errors | 
           % {
               if($SingleLineMessageOutputMode)
                 {
                   if($_.Procedure) 
                     {
                       $pr_str = " Procedure=$($_.Procedure)) "
                     } 
                   else 
                     { 
                       $pr_str=''
                     }
                   $MessageOut = @"
 $((Get-Date).ToString('yyyy.MM.dd HH:mm:ss')) SQL_ERR: No=$($_.Number) State=$($_.State) Class=$($_.Class) Line=$($_.LineNumber.ToString() + $pr_str) Message: $($_.Message) 
        
 "@              
                 }
               else
                 {
                   if($_.Procedure) 
                     {
                       $pr_str = @" 
 
    Procedure: $($_.Procedure)
 "@
                     } 
                   else 
                     { 
                       $pr_str=''
                     }        
                   $MessageOut = @"
 
 $((Get-Date).ToString('yyyy.MM.dd HH:mm:ss')) SQL_ERR:
       Number: $($_.Number)
        State: $($_.State)
        Class: $($_.Class)
       Server: $($_.Server)
       LineNo: $($_.LineNumber) $pr_str
      Message: $($_.Message) 
 "@                  
                 }
       
                 { 
                   Write-Host "$((Get-Date).ToString('yyyy.MM.dd HH:mm:ss')) SQL_MSG: $($_.Message)"
                   if($LogInfoMessagesToFile)
                     {
                       "$((Get-Date).ToString('yyyy.MM.dd HH:mm:ss')) SQL_MSG: $($_.Message)" | Out-File "$Path.OUT.txt" -Append
                     }
                 } 
               else 
                 { 
                   Write-Host $MessageOut 
                   if($LogInfoMessagesToFile)
                     {
                       $MessageOut | Out-File "$Path.OUT.txt" -Append
                     }
                 }
               }
         }
 
     $SmoServer.ConnectionContext.add_InfoMessage($SqlClientMessageHandler)
     $SmoDatabase = $SmoServer.Databases.Item($Database)
   
     if(!$SmoDatabase)
       {
         Write-Error -Message "Unable to reach database: $Database"
       }  
   
     $results  = $SmoDatabase.ExecuteWithResults($Batch)
   
     [Int]$i = 1
     $results[0].Tables | 
       % {
           $thisResultName = "ResultSets$($i.ToString('00'))"
     
           if($DisplayResultsToOGV)
             { 
               $_ | ogv -Wait -Title $thisResultName
             }
 
           if($DisplayResultsToHost)
             { 
               $thisResultName | oh
               $_ | ft -AutoSize | oh
             }
   
           if($ExportResultsAsCsv)
             {
               $_ | Export-Csv -NoTypeInformation -Force -Path "$Path.OUT.$thisResultName.csv"
             }
           $i++
         }  
     return $results
   }
`

