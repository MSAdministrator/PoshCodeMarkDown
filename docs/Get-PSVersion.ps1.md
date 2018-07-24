---
Author: powershell jedi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 959
Published Date: 2009-03-17t05
Archived Date: 2015-05-04t21
---

# get-psversion - 

## Description

simple function to get powershell version

## Comments



## Usage



## TODO



## function

`get-psversion`

## Code

`#
 #
 Set-Alias Ver Get-PSVersion
 function Get-PSVersion
 {
 [string]$Major = ($PSVersionTable).PSVersion.Major
 [string]$Minor = ($PSVersionTable).PSVersion.Minor
 [string]$Out = $Major + '.' + $Minor
 Write-Output $Out
 }
`

