---
Author: scott percy
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2986
Published Date: 2013-10-06t06
Archived Date: 2013-01-11t06
---

# start-mytranscript - 

## Description

this script, given a root path, will start a transcript with a unique and standardized file name.  you can throw a call to this at the end of your profile, and you’ll always have a transcript of every session (if the host you’re using supports starting a transcript).

## Comments



## Usage



## TODO



## function

`start-mytranscript`

## Code

`#
 #
 function Start-MyTranscript
 {
 	Param
 	(
 		[Parameter(Mandatory=$true,Position=0)]
 		[string] $Path
 	
 	Process
 	{
 		$fileextension = "txt"
 		$filenamePrefix = [System.DateTime]::Now.ToString("yyyy-MM-dd")
 		$existingFiles = Get-ChildItem (Join-Path $Path "$filenamePrefix.*.Transcript.$fileextension") | Sort-Object Name
 		if($existingFiles)
 		{
 			if($existingFiles.Count -gt 1)
 			{
 				$existingFilename = $existingFiles[-1].Name
 			else
 			{
 				$existingFilename = $existingFiles.Name
 			
 			$fileNumber = $existingFilename.Substring($filenamePrefix.Length + 1)
 			$fileNumber = $fileNumber.Substring(0, $fileNumber.IndexOf("."))
 			[int] $iFileNumber = 9999
 			if([Int32]::TryParse($fileNumber, [ref] $iFileNumber))
 			{
 				$iFileNumber++
 		else
 		{
 			$iFileNumber = 1
 		
 		if($iFileNumber)
 		{
 			$fileNumber = $iFileNumber.ToString("0000")
 			$filename = "$filenamePrefix.$fileNumber.Transcript.$fileextension"
 
 			if(!(Test-Path $Path))
 			{
 				New-Item $Path -Type Directory -Force
 			try
 			{
 				Start-Transcript -Path (Join-Path $Path $fileName) -Append -Force -NoClobber
 			catch
 			{
 				Write-Host "Unable to start transcript"
`

