---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5530
Published Date: 2015-10-21t03
Archived Date: 2015-01-31t20
---

# delete files per folder - 

## Description

another delete files script

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 
 .SYNOPSIS
 Deletes files in directory based on age measured in days
 
 .DESCRIPTION
 Delete files in folder with use of regular filter, either recursive or not.
 
 .PARAMETER DelFilter 
 Provide a filter like "*.txt" or "mylogs*"
 
 .PARAMETER DelPath
 The directory where files are to be deleted from, use the Recurse switch to delete from subfolder as well
 
 .EXAMPLE
 Remove-Files -delpath "C:\temp" -delfilter "Whatever-*" -fileage "30" -LogPath "C:\temp"
 
 .NOTES
 Instead of simply using a gci -Path -Filter -Recurse | Remove-Item I wanted a clean output per folder
 Enable debug mode, to write delete actions to the log file without actually deleting the files
 
 #>
 
 Param(
 $DelPath = $(throw "Provide path to delete files from"),
 $DelFilter = $(throw "Provide a filter like *.txt or mylogs*"),
 [int] $FileAge = $(throw "number of days to keep, set it to 0 for all files"),
 $LogPath = $($Env:windir),
 [switch] $Recurse, 
 [switch] $Debug
 )
 "Path: {0}" -f $DelPath
 "DelFilter: {0}" -f $DelFilter
 "Age: {0}" -f $FileAge
 "LogPath: {0}" -f $LogPath
 $LogFile = $LogPath + "\DelFiles" + ".log"
 $global:FileCount = 0
 
 Function WriteLog ($Output){
 	Write-output $Output | Out-File -Append -FilePath $LogFile
 	}
 Function DeleteFiles ($DelFolder){
 	foreach ($Item in Get-ChildItem -Force -Path $DelFolder -Filter $DelFilter){
 		if ($Item.CreationTime -lt ($(Get-Date).AddDays(-$FileAge))){
 			if (-not $Debug) { Remove-Item $Item.FullName}
 			WriteLog "`t $Item"
 			$global:FileCount = $global:FileCount +1
 		}
 	}
 }
 WriteLog "Deleting file(s) older than $FileAge day(s) at $DelPath"
 $Date = Get-Date
 WriteLog "Begin of operation at: $Date"
 WriteLog $Delpath
 DeleteFiles $Delpath
 if ($Recurse){
 	foreach ($Folder in (gci -Path $DelPath -recurse:$Recurse | ?{$_.PSIsContainer})){
 		$NrFiles = @(Get-ChildItem -Force -Path $Folder.Fullname -Filter $DelFilter).count
 		if ( $NrFiles -gt 0){
 			WriteLog $Folder.Fullname
 			DeleteFiles $Folder.Fullname
 		}
 	}
 }
 WriteLog "$global:FileCount file(s) deleted successfully"
 $Date = Get-Date
 WriteLog "End of operation at: $Date"
`

