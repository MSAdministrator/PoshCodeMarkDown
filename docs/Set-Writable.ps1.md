---
Author: keith hill
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1389
Published Date: 2010-10-12t15
Archived Date: 2017-04-08t23
---

# set-writable - 

## Description

from pscx, this filter can be used to change a file’s read only status.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param($Path, [switch]$PassThru, [switch]$Verbose)
 
 begin {
 	if ($args -eq '-?') {
 		""
 		"Usage: Set-Writable [[-Path] <object>] [[-PassThru] <switch>] [[-Verbose] <switch>]"
 		""
 		"Parameters:"
 		"    -Path     : Path of files or FileInfo objects to make writable"
 		"    -PassThru : Pass the input object down the pipeline"
 		"    -Verbose  : Display verbose information"
 		"    -?        : Display this usage information"
 		""
 		return
 	}
 
 	if ($verbose) { $VerbosePreference = 'Continue' }
 	
 	function MakeFileWritable($files) {
 		foreach ($file in @($files)) {
 			if ($file -is [System.IO.FileInfo]) {
 				if ($verbose) {
 					Write-Verbose "Set-Writable processing $file"
 				}
 				$file.IsReadOnly = $false
 				if ($passThru) {
 					$file
 				}
 			}
 			elseif ($file -is [string]) {
 				$rpaths = @(Resolve-Path $file -ea SilentlyContinue)
 				foreach ($rpath in $rpaths) {
 					if ($verbose) {
 						Write-Verbose "Set-Writable processing $rpath"
 					}
 					$fileInfo = New-Object System.IO.FileInfo $rpath
 					$fileInfo.IsReadOnly = $false
 					if ($passThru) {
 						$file
 					}
 				}
 			}
 			else {
 				Write-Error "Expected type FileInfo or String, got $($file.psobject.typenames[0]) instead."
 			}
 		}
 	}
 }
 
 process {
 	if ($_) {
 		MakeFileWritable $_
 	}
 }
 
 end {
 	if ($Path) {
 		MakeFileWritable $Path
     }
 }
`

