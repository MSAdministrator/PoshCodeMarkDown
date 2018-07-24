---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5167
Published Date: 2016-05-20t05
Archived Date: 2016-09-20t12
---

# get-wmiversions - 

## Description

use this script to detect installed .net versions on a remote server using wmi. requires credentials and a computername.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param ( $Credential, $ComputerName )
 
 
 $query = "select name from win32_directory where name like 'c:\\windows\\microsoft.net\\framework\\v%'"
 
 $res = Get-WmiObject -query $query -Credential $Credential -ComputerName $ComputerName | ForEach-Object {
 
 $prop = @{
 	ComputerName	= $ComputerName
 	V1Present		= &{ if ( $res | Where-Object { $_.Major -eq 1 -and $_.Minor -eq 0 } ) { $true } }
 	V1_1Present		= &{ if ( $res | Where-Object { $_.Major -eq 1 -and $_.Minor -eq 1 } ) { $true } }
 	V2Present		= &{ if ( $res | Where-Object { $_.Major -eq 2 -and $_.Minor -eq 0 } ) { $true } }
 	V3_5Present		= &{ if ( $res | Where-Object { $_.Major -eq 3 -and $_.Minor -eq 5 } ) { $true } }
 	VersionDetails	= $res
 }
 
 New-Object PSObject -Property $prop
`

