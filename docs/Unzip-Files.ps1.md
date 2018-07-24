---
Author: jayneticmuffin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5668
Published Date: 2017-01-07t14
Archived Date: 2017-04-17t14
---

# unzip files - 

## Description

wanted to create an unzip function for single files with an option to delete the originating zip, and also have a function for scanning through a folder structure recursively and providing the same functionality… and here it is.

## Comments



## Usage



## TODO



## function

`unzip-file`

## Code

`#
 #
 function Unzip-File {
 	param (
 		[parameter(mandatory=$true][ValidateNotNullOrEmpty()]$fileName,
 		$DeleteSource = $false
 	)
 	$fileInfo = Get-Item -Path $FileName
 	$appName = New-Object -ComObject Shell.Application
 	$zipName = $appName.NameSpace($fileInfo.FullName)
 	$extPath = $fileInfo.Directory.FullName + '\' + $fileInfo.BaseName
 	$null = New-Item -Path $extPath -ItemType Directory -Force
 	$dstFolder = $appName.NameSpace($extPath)
 	$dstFolder.Copyhere($zipName.Items())
 	If ($DeleteSource) {Remove-Item -Path $fileInfo.FullName}
 }
 function Unzip-MultipleFiles {
 	param (
 		[parameter(mandatory=$true)][ValidateNotNullOrEmpty()][string]$Path,
 		$DeleteSource = $false
 	)
 	$Files = Get-ChildItem -Path $Path -Recurse -Include '*.zip' | Select FullName,Directory,BaseName
 	$Files | % {
 		Unzip-File -FileName $_.FullName
 		If ($DeleteSource) {Remove-Item -Path $_.FullName}
 	}
 }
`

