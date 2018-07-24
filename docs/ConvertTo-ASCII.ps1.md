---
Author: levon becker
Publisher: 
Copyright: 
Email: 
Version: 1.001
Encoding: ascii
License: cc0
PoshCode ID: 4417
Published Date: 2013-08-25t13
Archived Date: 2013-09-01t00
---

# convertto-ascii - 

## Description

here is a quick script i created to convert files to ascii character set. this came about because i found out-file was creating files as unicode and add-content was writing ascii (even though the help says it defaults to unicode). so writing to the files with add-content would use the wrong encoding and add garbage to the log files. i changed all my code to include the -encoding ascii parameter, but had thousands of files encoded as unicode. (note

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
 	Automation script for file character set format conversion to ASCII encoding.
 .DESCRIPTION
 	Script for automating the conversion of files to ASCII format.  Good for if you've used Out-File without specifying the
 	encoding. Which would make a Unicode formatted file by default.  If you then used Add-Content to append data to that
 	file it would have garbage characters because even though it states it defaults to Unicode it will write ASCII/ANSI.
 .INPUTS
 	Full Path to files to convert.
 .OUTPUTS
 	Replacement files in ASCII format.
 .NOTES
 	AUTHOR:     Levon Becker
 	TITLE:      ConvertTo-Ascii
 	VERSION:    1.001
 	DATE:       4/13/2011
 	ENV:        Powershell v2
 	TOOLS:      PowerGUI Script Editor, RegexBuddy
 .EXAMPLE
 	.\ConvertTo-Ascii -path "C:\Folder1\Folder2\*" -include "*.txt"
 	Will convert all of the files with a TXT file extension in the directory named Folder2.
 .EXAMPLE
 	.\ConvertTo-Ascii -path "C:\Folder1\Folder2\*" -include "filename.txt"
 	Will convert a single file named filename.txt in the directory named Folder2
 .EXAMPLE
 	.\ConvertTo-Ascii -path "C:\Folder1\Folder2\*" -include "*.txt" -recursive $true
 	Will convert all of the files with a TXT file extension in the directory named Folder2 and any subdirectories.
 .EXAMPLE
 	.\ConvertTo-Ascii -path "C:\Folder1\Folder2\*" -include "filename.txt" -recursive $true
 	Will convert a single file named filename.txt in the directory named Folder2 and any subdirectories.
 .PARAMETER path
 	Full path to file/s to process. Standard wild cards are excepted. Be sure to put in quotation marks.
 	It should work without a trailing backslash or asterisk, but it's recommended to simply include.
 	See examples
 .PARAMETER include
 	File name filtering option. Standard wild cards are excepted. Be sure to put in quotation marks.
 .PARAMETER recursive
 	Specifies if the script should look in sub-folders off the path provided. Default is False (Do not do recursive).
 	Use -recursive $true to enable.
 .LINK
 #>
 
 
 
 <#
 04/12/2011: Created
 04/13/2011: Added time-stamp capture and write in order to keep the replacement files with the same time information as the original.
 			Cleaned up comments and regions.
 #>
 
 
 
 <#
 http://www.powergui.org
 http://www.regexbuddy.com
 #>
 
 
 
 [CmdletBinding()]
     Param (
         [parameter(Mandatory=$true)][string]$path,
 		[parameter(Mandatory=$true)][string]$include,
 		[parameter(Mandatory=$false)][string]$recursive = "$false"
        )
 
 
 
 $scriptver = '1.001'
 $originaltitle = $Host.UI.RawUI.WindowTitle
 $title = "File Convert to ASCII Script v$scriptver by Levon Becker"
 $count = $null
 $file = $null
 
 
 Clear
 $Host.UI.RawUI.WindowTitle = $title
 
 
 If ((Test-Path -Path $path) -eq $false) {
 	Write-Host "ERROR:	PATH NOT FOUND ($path)" -BackgroundColor Red -ForegroundColor White
 	Break
 }
 
 
 
 $regex = '\b\\\*'
 If ($path -notmatch $regex) {
 	$path = $path + '\*'
 }
 If ($recursive -eq $true) {
 	$filelist = Get-ChildItem -path $path -Include $include -Recurse
 }
 Else {
 	$filelist = Get-ChildItem -path $path -Include $include
 }
 
 Foreach ($file in $filelist) {
 	$count++
 	
 	$fullname = ($file.FullName)
 	$creationtime = ($file.CreationTime)
 	$creationtimeutc = ($file.CreationTimeUtc)
 	$lastwritetime = ($file.LastWriteTime)
 	$lastwritetimeutc = ($file.LastWriteTimeUtc)
 	$lastaccesstime = ($file.LastAccessTime)
 	$lastaccesstimeutc = ($file.LastAccessTimeUtc)
 	
 	$content = Get-Content $fullname
 	
 	Out-File -FilePath $fullname -Encoding ASCII -Force -InputObject $content
 	
 	Set-ItemProperty -Path $fullname -Name CreationTime ($creationtime)
 	Set-ItemProperty -Path $fullname -Name CreationTimeUtc ($creationtimeutc)
 	Set-ItemProperty -Path $fullname -Name LastWriteTime ($lastwritetime)
 	Set-ItemProperty -Path $fullname -Name LastWriteTimeUtc ($lastwritetimeutc)
 	Set-ItemProperty -Path $fullname -Name LastAccessTime ($lastaccesstime)
 	Set-ItemProperty -Path $fullname -Name LastAccessTimeUtc ($lastaccesstimeutc)
 
 	Write-Host $file
 
 }
 
 
 
 Write-Host '----------------------------------------------------------------------'
 Write-Host ''
 Write-Host "Total Converted: " -ForegroundColor Green -NoNewline
 Write-Host $count
 Write-Host ''
 Write-Host '-------------' -ForegroundColor Green
 Write-Host '| COMPLETED |' -ForegroundColor Green
 Write-Host '-------------' -ForegroundColor Green
 Write-Host ''
 
 
 $Host.UI.RawUI.WindowTitle = $title + ' (COMPLETED)'
 Sleep -Seconds 1
 $Host.UI.RawUI.WindowTitle = $originaltitle
`

