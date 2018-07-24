---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1620
Published Date: 
Archived Date: 2010-02-26t15
---

# purgefiles script. - 

## Description

recursively remove files with given extension and maximum age from a given path.

## Comments



## Usage



## TODO



## script

`delete-file`

## Code

`#
 #
 <#
 .SYNOPSIS
 	PurgeFiles - recursively remove files with given extension and maximum age from a given path.
 .DESCRIPTION
 	Read the synopsis
 	Example
 		PurgeFiles.psq -path C:\temp -ext .tmp -max 24
 .EXAMPLE
 	PurgeFiles.psq -path C:\temp -ext .tmp -max 24
 #>
 
 param(
 	 [Parameter(Mandatory=$true)][string] $path
 	,[Parameter(Mandatory=$true)][string] $extension
 	,[Parameter(Mandatory=$true)][int] $maxHours
 	,[switch] $deleteMode = $false
 	,[switch] $listMode = $false
 )
 
 function delete-file([string]$path, [string]$extension, [datetime]$oldestAllowed, [bool] $deleteMode, [bool] $listMode)
 {
 	$filesToRemove = Get-Childitem $path -recurse |
 		?{	!$_.PSIsContainer -and
 			($_.LastWriteTime -lt $oldestAllowed) -and
 			($_.Extension -eq $extension)
 		}
 	if ($listMode -and $filesToRemove) {
 		$filesToRemove | %{write-host "FILE: $($_.LastWriteTime) ""$($_.FullName)""`r`n"}
 	}
 	if ($deleteMode -and $filesToRemove) {
 		write-host "Removing Files...`r`n"
 		$filesToRemove | remove-item -force
 	}
 }
 
 $oldestAllowed = (get-date).AddHours(-$maxHours)
 
 if (-not $deleteMode) {
 	write-host "Running in trial mode, no files will be deleted.`r`n"
 }
 delete-file $path $extension $oldestAllowed $deleteMode $listMode
`

