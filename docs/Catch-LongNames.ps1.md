---
Author: mark ince
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5141
Published Date: 2016-05-04t07
Archived Date: 2016-01-21t15
---

# catch-longnames - 

## Description

function to find long filenames and the path they reside in. uses get-childitem and one instance of cmd.exe dir.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Catch-LongNames
 <#
 .SYNOPSIS
 This function stores the error from Get-ChildItem when it encounters files longer than the 260 Character Limit.
 .DESCRIPTION
 This function stores the error from Get-ChildItem when it encounters files longer than the 260 Character Limit.
 This functin will produce a lot of error messages on the shell. This is normal behavior (After all, this funciton relies on there being errors. If you want to see them, comment out the $ErrorActionPreference = "SilentlyContinue"'.
 .EXAMPLE
 Catch-LongNames C:
 Outputs an array of files that exceeded the character limit for Drive C:\.
 .EXAMPLE
 Catch-LongNames C: -Export
 Outputs an array of files that exceeded the character limit for Drive C:\ and outputs them to a text file in the current PowerShell directory. .\
 .EXAMPLE
 Catch-LongNames C: -Find
 Outputs an array of paths that contain files that exceed the character limit and an array of files in those paths that are the offenders. This switch uses CMD.exe and the creation of a text file. (Dir outputs even the files that are too long) Dir to a variable which is compared to a variable from the output of Get-ChildItem in the same directory. These two variables are then sent to Compare-Object.
 .EXAMPLE
 Catch-LongNames C: -Export -Find
 Outputs an array of paths that contain files that exceed the character limit and an array of files in those paths that are the offenders. This switch uses CMD.exe and the creation of a text file. (Dir outputs even the files that are too long) Dir to a variable which is compared to a variable from the output of Get-ChildItem in the same directory. These two variables are then sent to Compare-Object. All of the output on the shell is also sent to two text files in $env:userprofile direcory, named LongPaths.txt and TooLongFiles.txt respectively.
 .PARAMETERS
 -Path
 .Author
 Mark R. Ince mrince@outlook.com
 #>
 {
 	[CmdletBinding()]
 	param
 	(
 		[Parameter(Mandatory=$True,
 		ValueFromPipeline=$True,
 		Position=0)]
 		[string[]]$Path,
 		[Parameter(Mandatory=$False)]
 		[switch]
 		$Export,
 		[Parameter(Mandatory=$False)]
 		[switch]
 		$Find		
 	)
 	BEGIN
 	{
 	}
 	PROCESS
 	{
 			Get-ChildItem -path $path -r * | Out-Null
 	}
 	END
 	{
 		{
 			If ($Export -eq $True)
 			{
 			}			
 		}
 
 		If ($Find -eq $True)
 		{
 			ForEach ($resfile in $longfiles)
 			{
 				$Locate = $env:userprofile
 				Set-Location $resfile				
 				$check = cmd /C "dir /b ""$resfile"" /b"
 				$validfiles = Get-ChildItem -r
 				$toolong = (Compare-Object $validfiles $check).InputObject
 				$files += $toolong
 				Set-Location $Locate
 			}
 			If ($Export -eq $True)
 			{
 				$files | Out-File "$env:userprofile\TooLongFiles.txt"
 			}
 			$files
 		}
 	}
 }
`

