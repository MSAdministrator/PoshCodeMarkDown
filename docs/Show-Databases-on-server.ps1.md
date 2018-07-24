---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5984
Published Date: 2016-08-24t18
Archived Date: 2016-05-17t10
---

# show databases on server - show-databasesonserver.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-databasesonserver`

## Code

`#
 #
  #############################################################################################
 #
 #
 
 
 Function Show-DatabasesOnServer ([string]$Server)
 {
 $srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
 
 Write-Host " The Databases on $Server Are As Follows"
 $srv.databases| Select Name
 }
`

