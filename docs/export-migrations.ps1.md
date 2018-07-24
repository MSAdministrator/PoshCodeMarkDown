---
Author: steve
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5645
Published Date: 2015-12-15t16
Archived Date: 2015-02-17t17
---

# export migrations - 

## Description

export all mailbox migration stats.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Migrations = get-moverequest
 
 foreach ($migration in $migrations) {Get-MoveRequestStatistics $migration.alias -IncludeReport | select displayname,status,starttimestamp,completiontimestamp,overallduration,totalmailboxsize,totalmailboxitemcount} | export-csv -Delimiter ';' -NoTypeInformation -Force -Path 'C:\Users\Tester\Desktop\Migrationstats.csv'
`

