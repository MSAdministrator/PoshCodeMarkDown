---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 925
Published Date: 
Archived Date: 2017-04-30t10
---

#  - 

## Description

update sharepoint search scopes, useful when revving the scope definitions.

## Comments



## Usage



## TODO



## function

`update-scopes`

## Code

`#
 #
 function Update-Scopes($siteUrl)
 {
 	[void][reflection.assembly]::Loadwithpartialname("Microsoft.SharePoint") | out-null
 	[void][reflection.assembly]::Loadwithpartialname("Microsoft.office.server.search") | out-null
 	
 	$s = [microsoft.sharepoint.spsite]$siteUrl
 	$sc = [microsoft.office.server.servercontext]::GetContext($s)
 	$search = [Microsoft.Office.Server.Search.Administration.SearchContext]::GetContext($sc)
 	$scopes = [microsoft.office.server.search.administration.scopes]$search
 	$scopes.StartCompilation()
 	while ($scopes.CompilationPercentComplete -lt 100) { sleep -seconds 3; write-host "$($scopes.CompilationPercentComplete)% complete" }
 }
 
 Update-Scopes -siteUrl "http://dev/sites/MySPSite/"
`

