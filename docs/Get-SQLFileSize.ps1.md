---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2537
Published Date: 2011-03-04t19
Archived Date: 2011-04-15t09
---

# get-sqlfilesize - 

## Description

this function will allow you to query any server hosting sql and return the file sizes for the each database file(mdf) and transaction log file(ldf). this does not return back the file locations for each database. i have tested this on sql 2000, 2005 and 2008.

## Comments



## Usage



## TODO



## function

`get-sqlfilesize`

## Code

`#
 #
 Function Get-SQLFileSize { 
 .SYNOPSIS   
     Retrieves the file size of a MDF or LDF file for a SQL Server 
 .DESCRIPTION 
     Retrieves the file size of a MDF or LDF file for a SQL Server 
 .PARAMETER Computer 
     Computer hosting a SQL Server 
 .NOTES   
     Name: Get-SQLFileSize 
     Author: Boe Prox 
     DateCreated: 17Feb2011         
 .EXAMPLE   
 Get-SQLFileSize -Computer Server1 
  
 Description 
 ----------- 
 This command will return both the MDF and LDF file size for each database on Server1 
 .EXAMPLE   
 Get-SQLFileSize -Computer Server1 -LDF 
  
 Description 
 ----------- 
 This command will return LDF file size for each database on Server1 
 Description 
 ----------- 
 This command will return both the MDF and LDF file size for each database on Server1 
 .EXAMPLE   
 Get-SQLFileSize -Computer Server1 -MDF 
  
 Description 
 ----------- 
 This command will return MDF file size for each database on Server1 
  
 #>  
 [cmdletbinding( 
     DefaultParameterSetName = 'Default', 
     ConfirmImpact = 'low' 
 )] 
     Param( 
         [Parameter( 
             Mandatory = $True, 
             Position = 0, 
             ParameterSetName = '', 
             ValueFromPipeline = $True)] 
             [string[]]$Computer, 
         [Parameter( 
             Mandatory = $False, 
             Position = 1, 
             ParameterSetName = '', 
             ValueFromPipeline = $False)] 
             [switch]$Mdf, 
         [Parameter( 
             Mandatory = $False, 
             Position = 2, 
             ParameterSetName = '', 
             ValueFromPipeline = $False)] 
             [switch]$Ldf                         
         ) 
 Begin { 
     If (!($PSBoundParameters.ContainsKey('Mdf')) -AND !($PSBoundParameters.ContainsKey('Ldf'))) { 
         Write-Verbose "MDF or LDF not selected, scanning for both file types" 
         $FileFlag = $True
         $Flag = $False 
         } 
     Write-Verbose "Creating holder for data" 
     $report = @()
     } 
 Process {     
     ForEach ($comp in $Computer) { 
         Write-Verbose "Testing server connection" 
         If (Test-Connection -count 1 -comp $comp -quiet) { 
             If ($PSBoundParameters.ContainsKey('Mdf') -OR $FileFlag) { 
                 Write-Verbose "Looking for MDF file sizes"  
                     Try { 
                         Write-Verbose "Attempting to retrieve counters from server" 
                         $DBDataFile = Get-Counter -Counter '\SQLServer:Databases(*)\Data File(s) Size (KB)' -MaxSamples 1 -comp $comp -ea stop 
                         $DBDataFile.CounterSamples | % { 
                             If ($_.InstanceName -ne "_total") { 
                                 $temp = "" | Select Computer, Database, FileType, Size_MB 
                                 $temp.Computer = $comp 
                                 $temp.Database = $_.InstanceName 
                                 $temp.FileType = 'MDF' 
                                 $temp.Size_MB = $_.CookedValue/1000 
                                 $report += $temp 
                                 } 
                             } 
                         } 
                     Catch { 
                         $Flag = $True 
                         }
                     If ($Flag) {                 
                         Try { 
                             Write-Verbose "Attempting to retrieve counters from server" 
                             $DBDataFile.CounterSamples | % { 
                                 If ($_.InstanceName -ne "_total") { 
                                     $temp = "" | Select Computer, Database, FileType, Size_MB 
                                     $temp.Computer = $comp 
                                     $temp.Database = $_.InstanceName 
                                     $temp.FileType = 'MDF' 
                                     $temp.Size_MB = $_.CookedValue/1000 
                                     $report += $temp 
                                     } 
                                 }             
                             } 
                         Catch { 
                             Write-Warning "$($Comp): Unable to locate Database Counters or Database does not exist on this server"
                             Break
                             }
                         }
                     Else {
                         Write-Warning "$($Comp): Unable to locate Database Counters or Database does not exist on this server"
                         Break
                         } 
                 } 
             If ($PSBoundParameters.ContainsKey('Ldf') -OR $FileFlag) {  
                 Write-Verbose "Looking for LDF file sizes"                
                     Try { 
                         Write-Verbose "Attempting to retrieve counters from server" 
                         $DBDataFile = Get-Counter -Counter '\SQLServer:Databases(*)\Log File(s) Size (KB)' -MaxSamples 1 -comp $comp -ea stop 
                         $DBDataFile.CounterSamples | % { 
                             If ($_.InstanceName -ne "_total") { 
                                 $temp = "" | Select Computer, Database, FileType, Size_MB 
                                 $temp.Computer = $comp 
                                 $temp.Database = $_.InstanceName 
                                 $temp.FileType = 'LDF' 
                                 $temp.Size_MB = $_.CookedValue/1000 
                                 $report += $temp 
                                 } 
                             } 
                         } 
                     Catch { 
                         $Flag = $True  
                         }
                     If ($flag) {                 
                         Try { 
                             Write-Verbose "Attempting to retrieve counters from server" 
                             $DBDataFile.CounterSamples | % { 
                                 If ($_.InstanceName -ne "_total") { 
                                     $temp = "" | Select Computer, Database, FileType, Size_MB 
                                     $temp.Computer = $comp 
                                     $temp.Database = $_.InstanceName 
                                     $temp.FileType = 'LDF' 
                                     $temp.Size_MB = $_.CookedValue/1000 
                                     $report += $temp 
                                     } 
                                 }             
                             } 
                         Catch { 
                             Write-Warning "$($Comp): Unable to locate Database Counters or Database does not exist on this server"
                             Break
                             }
                         }
                     Else {
                         Write-Warning "$($Comp): Unable to locate Transaction Log Counters or Database does not exist on this server"
                         Break
                         } 
                     } 
             } 
         Else { 
             Write-Warning "$($Comp) not found!" 
             }                
         }         
     } 
 End { 
     Write-Verbose "Displaying output" 
     $report 
     }                 
 }
`

