---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1551
Published Date: 
Archived Date: 2009-12-25t16
---

#  - 

## Description

copy from source to destination and rename existing destination files to .old, if .old exists replace it.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param([Parameter(Mandatory=$true)][string]$Path,[Parameter(Mandatory=$true)][string]$Destination)
 
 Get-ChildItem -Path $Path | Where-Object { !$_.PSIsContainer } | foreach {
 	$Target = Join-Path -Path $Destination -ChildPath (Split-Path -Leaf $_)
 	if ( Test-Path -Path $Target -PathType Leaf ) {
 		$PrevTargetBkup=([System.IO.Path]::ChangeExtension($Target, ".old"))
 		if ( Test-Path -Path $PrevTargetBkup -PathType Leaf ) {
 			Remove-Item -Force -Path $PrevTargetBkup
 		}
 		Rename-Item -Path $Target -NewName ([System.IO.Path]::ChangeExtension($Target, ".old"))
 	}
 	Copy-Item -Path $_ -Destination $Target
 }
`

