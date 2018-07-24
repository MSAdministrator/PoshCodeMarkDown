---
Author: redsolar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5241
Published Date: 2014-06-14t06
Archived Date: 2014-07-01t09
---

# recursive file update - 

## Description

asks for full path of source file and target directory to recursively update all instances of file within target directory and its sub-directories. there is also an optional prompt to configure whether you want to be prompted for every file copy. no parameters are passed to the script. run it, type/paste in your paths, and choose your overwrite confirmation choice.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Write-Host
 
 $SourceFilePath = Read-Host "Source File Path"
 
 $TargetDirectory = Read-Host "Target Directory"
 
 Write-Host
 
 $Prompt = Read-Host "Prompt before overwriting (Y or N)?"
 
 $Prompt = $Prompt.ToUpper()
 if ($Prompt -eq "" -or ($Prompt -ne "Y" -and $Prompt -ne "N")) {
 	$Prompt = "Y"
 }
 
 $SourceFileName = $SourceFilePath.Substring($SourceFilePath.LastIndexOf('\')+1)
 
 $SourceFileModified = (Get-Item $SourceFilePath).LastWriteTime
 
 Write-Host
 Write-Host "Source file modified date is $($SourceFileModified)"
 Write-Host
 
 $matches = Get-ChildItem -Path $TargetDirectory -Filter $SourceFileName -Recurse |
 	Where { $_.LastWriteTime -lt $SourceFileModified } |
 	Foreach-object {
 		if ($Prompt -eq "N") {
 			Copy-Item $SourceFilePath $_.FullName.Substring(0, $_.FullName.LastIndexOf('\'))
 			Write-Host "$($_.FullName) [$($_.LastWriteTime)] [updated]"
 		} else {
 			Write-Host "Match found > $($_.FullName) [$($_.LastWriteTime)]"
 			Copy-Item -Confirm $SourceFilePath $_.FullName.Substring(0, $_.FullName.LastIndexOf('\'))
 		}
 	}
 
 Write-Host
`

