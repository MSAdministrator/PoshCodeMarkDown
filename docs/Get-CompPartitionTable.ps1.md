---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4108
Published Date: 2013-04-15t20
Archived Date: 2013-05-09t11
---

# get-comppartitiontable - 

## Description

quick script to get compressed or partitioned sql server tables using sqlps provider

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 param($computer,$instance,$database)
 
 import-module sqlps -disablenamechecking
 
 $path = "SQLSERVER:\SQL\$($computer)\$($instance)\Databases\$($database)\Tables"
 SET-LOCATION $path
 get-childitem | where {$_.HasCompressedPartitions -or $_.IsPartitioned} | 
 select @{n='ServerInstance';e={"$computer\$instance"}},@{n='Database';e={$database}}, name, HasCompressedPartitions, IsPartitioned
 cd $home
`

