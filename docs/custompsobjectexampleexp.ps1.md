---
Author: redyey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5646
Published Date: 2015-12-15t16
Archived Date: 2015-02-17t23
---

# custompsobjectexampleexp - 

## Description

custompsobject

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $AllValues = @()
 $Values = @()
 
 $Migrations = Get-MoveRequest
 
 foreach ($Migration in $Migrations) 
 {
     $Values = Get-MoveRequestStatistics $Migration.alias -IncludeReport | Select-Object displayname,status,starttimestamp,completiontimestamp,overallduration,totalmailboxsize,totalmailboxitemcount
 
     $NewObject = [pscustomobject]@{
         Displayname = $Values.Displayname
         Status = $Values.Status
         StartTimeStamp = $Values.StartTimeStamp
         CompletionTimeStamp = $Values.Completiontimestamp
         OverallDuration = $Values.OverallDuration
         TotalMailboxSize = $Values.TotalMailBoxSize
         TotalMailboxItemCount = $Values.TotalMailboxItemCount
     }
     $Allvalues += $NewObject
 }
 
 $AllValues | Export-Csv -Delimiter ';' -NoTypeInformation -Force -Path 'C:\Users\Tester\Desktop\Migrationstats.csv'
`

