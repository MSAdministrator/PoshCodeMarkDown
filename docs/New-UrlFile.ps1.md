---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 407
Published Date: 2009-05-23t05
Archived Date: 2016-04-19t06
---

# new-urlfile - 

## Description

use this to create a .url file which can then be opened in your default browser using the invoke-item cmdlet.  usage

## Comments



## Usage



## TODO



## function

`new-urlfile`

## Code

`#
 #
 function New-UrlFile
 {
 	param( $URL = "http://www.google.com")
 	$UrlFile = [system.io.Path]::ChangeExtension([system.io.Path]::GetTempFileName(),".url")
 	$UrlFileContents = `
 		"[InternetShortcut]",
 		"URL=$URL"
 	Write-Host $URL
 	$UrlFileContents | Set-Content -Path $UrlFile
 	Get-Item $UrlFile
 }
`

