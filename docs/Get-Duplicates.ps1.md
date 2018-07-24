---
Author: dragonmc
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5239
Published Date: 2015-06-14t04
Archived Date: 2015-01-31t20
---

# get-duplicates - 

## Description

finds and reports duplicate files between two folders.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 . "$(Split-Path $MyInvocation.MyCommand.Path -Parent)\globalfunctions.ps1"
 
 [string]$Path1 = Get-SwitchValue $Args "path1"
 [string]$Path2 = Get-SwitchValue $Args "path2"
 
 if ($Path1 -eq "") {$Path1 = Read-Host "Enter Path 1"}
 
 if (-not [IO.Directory]::Exists($Path1)) {
 	$Path1 = [regex]::Replace($Path1, "\$", "")
 	$Parent = [IO.Path]::GetDirectoryName($Path1)
 	$Name = [IO.Path]::GetFileName($Path1)
 	if ($Dirs.Length -gt 0) {
 		$Path1 = $Dirs[0]
 		Write-Host "Using $Path1" -ForegroundColor "Gray"
 	} else {Write-Host "Path 1 is not valid." ; break}
 } else {"Path 1: $Path1"}
 
 if ($Path2 -eq "") {$Path2 = Read-Host "Enter Path 2"}
 
 if (-not [IO.Directory]::Exists($Path2)) {
 	$Path2 = [regex]::Replace($Path2, "\$", "")
 	$Parent = [IO.Path]::GetDirectoryName($Path2)
 	$Name = [IO.Path]::GetFileName($Path2)
 	if ($Dirs.Length -gt 0) {
 		$Path2 = $Dirs[0]
 		Write-Host "Using $Path2" -ForegroundColor "Gray"
 	} else {Write-Host "Path 2 is not valid." ; break}
 } else {"Path 2: $Path2"}
 
 $DuplicateFiles = @()
 $Folder1Paths = @()
 $Folder2Paths = @()
 
 $Size = 0
 Get-ChildItem $Path1 -Recurse |
 	Where-Object {$_.GetType().Name -eq "FileInfo"} |
 	ForEach-Object	-Begin {Write-Host "Getting list of all files in Path 1..." -NoNewline} `
 					-Process {$Folder1Paths += @($_.FullName.Replace($Path1, "").ToLower()); $Size += $_.Length} `
 					-End {Write-Host "Done ($(($Size / 1024) / 1024) MB found in $($Folder1Paths.Count) files.`)"}
 
 $Size = 0
 Get-ChildItem $Path2 -Recurse |
 	Where-Object {$_.GetType().Name -eq "FileInfo"} |
 	ForEach-Object	-Begin {Write-Host "Getting list of all files in Path 2..." -NoNewline} `
 					-Process {$Folder2Paths += @($_.FullName.Replace($Path2, "").ToLower()); $Size += $_.Length} `
 					-End {Write-Host "Done ($(($Size / 1024) / 1024) MB found in $($Folder2Paths.Count) files.`)"}
 
 Write-Host "Searching for duplicates..." -NoNewLine
 $DuplicateFiles = ($Folder1Paths | Where-Object {$Folder2Paths -contains $_})
 Write-Host "Done."
 
 if ($DuplicateFiles.Length -gt 0) {
 	$Count = 0
 	Set-Content	-Path "$(Split-Path $MyInvocation.MyCommand.Path -Parent)\$([IO.Path]::GetFileName($Path1))_Duplicates.txt" `
 				-Value "File list contains only files that are newer."
 	Write-Host "Listing newer files..."
 	foreach ($FileName in $DuplicateFiles) {
 		$File1 = New-Object System.IO.FileInfo("$Path1$FileName")
 		$File2 = New-Object System.IO.FileInfo("$Path2$FileName")
 		if ($File1.LastWriteTime -gt $File2.LastWriteTime) {
 			"$Path1$FileName" |
 				Add-Content -Path "$(Split-Path $MyInvocation.MyCommand.Path -Parent)\$([IO.Path]::GetFileName($Path1))_Duplicates.txt" -PassThru |
 				Write-Host
 		}
 		elseif ($File1.LastWriteTime -lt $File2.LastWriteTime) {
 			"$Path2$FileName" |
 				Add-Content -Path "$(Split-Path $MyInvocation.MyCommand.Path -Parent)\$([IO.Path]::GetFileName($Path1))_Duplicates.txt" -PassThru |
 				Write-Host
 		}
 		else {$Count += 1}
 	}
 	Write-Host "$Count of the duplicates have the same Modified date/time stamp." -ForegroundColor "Red"
 	Write-Host "$($DuplicateFiles.Count) total duplicates found." -ForegroundColor "Red"
 	Write-Host ""
 }
 else {
 	Write-Host "No duplicates found" -ForegroundColor "Green"
 	Write-Host ""
 }
 Remove-Variable DuplicateFiles
 Remove-Variable Folder1Paths
 Remove-Variable Folder2Paths
`

