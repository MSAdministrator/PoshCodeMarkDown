---
Author: andy schneider
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 536
Published Date: 2008-08-18t22
Archived Date: 2012-12-29t04
---

# set-webconfig - 

## Description

a function to set a sql connection string in a web.config file

## Comments



## Usage



## TODO



## function

`set-webconfigsqlconnectionstring`

## Code

`#
 #
 function Set-WebConfigSqlConnectionString {
 	param(  [switch]$help,
 	        [string]$configfile = $(read-host "Please enter a web.config file to read"),
 	        [string]$connectionString = $(read-host "Please enter a connection string"),
 	        [switch]$backup = $TRUE	
 		)
 	
 	$usage = "`$conString = `"Data Source=MyDBname;Initial Catalog=serverName;Integrated Security=True;User Instance=True`"`n"
 	$usage += "`"Set-WebConfigSqlConnectionString -configfile `"C:\Inetpub\wwwroot\myapp\web.config`" -connectionString `$conString"
 	if ($help) {Write-Host $usage;break}
 	
 	
 	$webConfigPath = (Resolve-Path $configfile).Path 
 	$backup = $webConfigPath + ".bak"
 
 	$xml = [xml](get-content $webConfigPath)
 
 	if ($backup) {$xml.Save($backup)}
 
 	$root = $xml.get_DocumentElement();
 	$root.connectionStrings.add.connectionString = $connectionString
 	$xml.Save($webConfigPath)
 	
 	}
`

