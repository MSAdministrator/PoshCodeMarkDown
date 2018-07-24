---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 934
Published Date: 2010-03-11t19
Archived Date: 2016-03-05t14
---

# copy-fileplus - 

## Description

(requires v2 ctp3)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <#
 .SYNOPSIS
 	Copies a file from one location to another while displaying a GUI progress window.
 .PARAMETER Path
 	Specifies the filename or FileInfo object representing file to be copied.  Right now, this must be fully-qualified, relative paths will produce an error.  Try it with Get-Item or Get-ChildItem, this works great.
 .PARAMETER Destination
 	Specifies the filename including path for resulting copy operation.
 .EXAMPLE
 	PS > Copy-FilePlus -Path c:\tmp\windows7.iso -Destination e:\tmp\windows7.iso
 .EXAMPLE
 	PS > Get-Item c:\tmp\windows7.iso | Copy-FilePlus -Destination e:\tmp\windows7.iso
 #>
 param (
 	[Parameter(
 		Mandatory = $true, 
 		ValueFromPipeline = $true
 	)]$Path,
 	[Parameter(Mandatory=$true)]
 	[string]
 	$Destination
 )
 try {
 	add-type -a microsoft.visualbasic
 	[Microsoft.VisualBasic.FileIO.FileSystem]::CopyFile(
 		$Path,
 		$Destination,
 		[Microsoft.VisualBasic.FileIO.UIOption]::AllDialogs,
 		[Microsoft.VisualBasic.FileIO.UICancelOption]::ThrowException
 	)
 } catch { $_ }
`

