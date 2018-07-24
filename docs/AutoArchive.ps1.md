---
Author: christophecremon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2563
Published Date: 2011-03-15t12
Archived Date: 2011-03-24t17
---

# autoarchive - 

## Description

autoarchive script

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 param (
 	[ValidateNotNullOrEmpty()]
 		[Parameter(
     		Mandatory = $true)
     	]
 			[string] $Source,
 		[int] $RetentionDays,
 		[array] $Include,
 		[array] $Exclude,
 		[switch] $Recurse )
 
 Import-Module PowerZip
 
 if ( -not ( Test-Path -Path "$Source" -ErrorAction SilentlyContinue ) )
 {
 	throw "ERROR : Source not found { $Source }"
 }
 
 $DirectoryTimeStamp = (Get-Date).ToString("yyyy\\MM")
 $ArchiveTimeStamp = (Get-Date).ToString("yyyyMMddHHmmss")
 if ( $Recurse -eq $true ) { $RecurseArgument = "-Recurse" }
 if ( $Include )
 {
 	$Include = $Include -join ","
 	$IncludeArgument = "-Include $Include"
 	$Source = $Source+"\*"
 }
 $Exclude += @("*.zip")
 $Exclude = $Exclude -join ","
 $ExcludeArgument = "-Exclude $Exclude"
 
 $GetCommand = "Get-ChildItem -Path '$Source' $IncludeArgument $ExcludeArgument $RecurseArgument"
 
 Invoke-Expression -Command $GetCommand | Where-Object { ( $_.LastWriteTime -lt (Get-Date).AddDays(-$RetentionDays) ) -and ( $_.psIsContainer -eq $false ) -and ( $_ -cnotmatch "\\_AutoArchive_\\" ) } | ForEach-Object {
 	$ArchiveDirectory = $_.DirectoryName
 	$ArchiveDirectory = "$ArchiveDirectory\_AutoArchive_\$DirectoryTimeStamp"
 	Write-Output "Moving { $($_.FullName) } to { $ArchiveDirectory } ..."
 	$DirectoryToZipArray += @($ArchiveDirectory)
 	if ( -not ( Test-Path -Path "$ArchiveDirectory" -ErrorAction SilentlyContinue ) )
 	{
 		New-Item -ItemType Directory -Path "$ArchiveDirectory" | Out-Null
 		if ( $? -ne $true )
 		{
 			$ErrorsArray += @("! Unable to create directory {$ArchiveDirectory}")
 		}
 	}
 	Move-Item -Path $_.FullName -Destination "$ArchiveDirectory" -Force -ErrorAction SilentlyContinue
 	if ( $? -ne $true )
 	{
 		$ErrorsArray += @("! Unable to move file {$($_.FullName)}")
 	}
 }
 
 foreach ( $DirectoryToZip in $DirectoryToZipArray | Sort-Object -Unique )
 {
 	Write-Output "Zipping { $DirectoryToZip } ..."
 	$ZipFile = "$DirectoryToZip\$ArchiveTimeStamp.zip"
 	$Zip = New-Zip -Source "$DirectoryToZip" -ZipFile "$ZipFile" -DeleteAfterZip -Exclude "*.zip"
 	if ( $? -ne $true )
 	{
 		$ErrorsArray += @("! Unable to zip directory {$DirectoryToZip}")
 	}
 }
 
 if ( $ErrorsArray )
 {
 	Write-Output "`n[ ERRORS OCCURED ]"
 	$ErrorsArray
 	return $false
 }
 else
 {
 	return $true
 }
`

