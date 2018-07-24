---
Author: sean wendt
Publisher: 
Copyright: 
Email: 
Version: 0.9
Encoding: ascii
License: cc0
PoshCode ID: 3944
Published Date: 2015-02-12t19
Archived Date: 2015-01-21t07
---

# reverse filename sequenc - 

## Description

this script will rename a sequenced set of files in a directory.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 
 $file_extension = Read-Host "Enter file extension, i.e. .jpg or .txt"
 $file_prefix = Read-Host "Enter file prefix, i.e. entering foobar will rename files to foobar001.ext, foobar002.ext, etc.."
 
 for ($ctr=$files.count; $ctr -gt 0 ; --$ctr)
 {
 	if (($idx -ge 0) -and ($idx -le 9))
 		{
 		Rename-Item -path $files[$ctr-1].name -newname ($file_prefix + '00000' + $idx++ + $file_extension)
 		}
 	elseif (($idx -ge 10) -and ($idx -le 99))
 		{
 		Rename-Item -path $files[$ctr-1].name -newname ($file_prefix + '0000' + $idx++ + $file_extension)
 		}
 	elseif (($idx -ge 100) -and ($idx -le 999))
 		{
 		Rename-Item -path $files[$ctr-1].name -newname ($file_prefix + '000' + $idx++ + $file_extension)
 		}
 	elseif (($idx -ge 1000) -and ($idx -le 9999))
 		{
 		Rename-Item -path $files[$ctr-1].name -newname ($file_prefix + '00' + $idx++ + $file_extension)
 		}
 	elseif (($idx -ge 10000) -and ($idx -le 99999))
 		{
 		Rename-Item -path $files[$ctr-1].name -newname ($file_prefix + '0' + $idx++ + $file_extension)
 		}
 }
`

