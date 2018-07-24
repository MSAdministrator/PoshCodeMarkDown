---
Author: ryan smith
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3836
Published Date: 2012-12-19t12
Archived Date: 2012-12-22t08
---

# write-scriptvariables - 

## Description

print variables defined in the script (excludes global vars)

## Comments



## Usage



## TODO



## function

`write-scriptvariables`

## Code

`#
 #
 function Write-ScriptVariables {
    $globalVars = get-variable -scope Global | % { $_.Name }
    Get-Variable -scope Script | Where-Object { $globalVars -notcontains $_.Name } | Where-Object { $_.Name -ne 'globalVars' } | Out-String	
 }
`

