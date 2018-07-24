---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3435
Published Date: 
Archived Date: 2012-06-06t05
---

#  - 

## Description

a function to get a sql connection string in a web.config file

## Comments



## Usage



## TODO



## function

`get-webconfigsqlconnectionstring`

## Code

`#
 #
 
 function Get-WebConfigSqlConnectionString 
 {
 param( [switch]$help, [string]$configfile = $(read-host "Please enter a web.config file to read"), 
 [string]$conname = $(read-host "Please enter connection name"));
 
   $usage = "Usage: Get-WebConfigSqlConnectionString -configfile c:\inetpub\wwwroot\web.config -conname 'ConName'";
 
   if ($help) {Write-Host $usage;break}
 
   $webConfigPath = (Resolve-Path $configfile).Path;
   $xml = [xml](get-content $webConfigPath);
   $root = $xml.get_DocumentElement();
   $connStrings = $root.connectionStrings;
   $addTag = $connStrings.add
   $conStringTag = $addTag | Where { $_.Name -eq $conname}
   return $conStringTag.connectionString
 }
`

