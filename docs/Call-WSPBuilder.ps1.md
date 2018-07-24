---
Author: peter
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1366
Published Date: 2009-10-02t12
Archived Date: 2013-11-16t08
---

# call wspbuilder - 

## Description

small snippet of my build script, i think this is a good template to use when calling the command-line wspbuilder.

## Comments



## Usage



## TODO



## function

`run-doscommand`

## Code

`#
 #
 function Run-DosCommand($program, [string[]]$programArgs)
 {
 	write-host "Running command: $program";
 	write-host " Args:"
 	0..($programArgs.Count-1) | foreach { Write-Host " $($_): $($programArgs[$_])" }
 	& $program $programArgs
 }
 
 $wspbuilder = Get-FullPath("tools\WSPBuilder.exe")
 function Run-WspBuilder($rootDirectory)
 {
 	pushd
 	cd $rootDirectory
 	Run-DosCommand -program $WSPBuilder -programArgs @("-BuildWSP", 
 		"true", 
 		"-OutputPath", 
 		(Get-FullPath 'deployment'), 
 		"-ExcludePaths",
 		(Join-Path -path (Get-FirstDirectoryUnderneathSrc).fullname -childPath "bin\Debug"))
 	popd
 }
`

