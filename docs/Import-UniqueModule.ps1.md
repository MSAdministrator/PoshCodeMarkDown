---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2303
Published Date: 
Archived Date: 2010-10-20t09
---

# import-uniquemodule - 

## Description

an attempt to resolve namespace clashes without overwriting functions …

## Comments



## Usage



## TODO



## function

`import-uniquemodule`

## Code

`#
 #
 function Import-UniqueModule { 
 
 param([Parameter(Mandatory=$true)][String]$ModuleName)
 
 $unique = [guid]::NewGuid().Guid -replace "-"
 Import-Module $ModuleName -Prefix $unique
 Get-Command -Module $ModuleName | 
    New-Alias -Name {$_.Name -replace $unique} -Value { "{0}/{1}" -f $_.ModuleName, $_.name }
 
 }
`

