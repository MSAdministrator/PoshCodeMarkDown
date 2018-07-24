---
Author: glenn
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5921
Published Date: 2015-07-07t14
Archived Date: 2015-07-10t13
---

# coping files ... - 

## Description

copy data between folders including a progressbar

## Comments



## Usage



## TODO



## function

`copy-data`

## Code

`#
 #
 function copy-data {
 	param($source, $dest)
 	$counter = 0
 	$files = Get-ChildItem $source -Force -Recurse
 	foreach($file in $files)
 		{
 		$status = "Copying file {0} of {1}: {2}" -f $counter, $files.count, $file.name
 		Write-Progress -Activity "Copying Files" -Status $status -PercentComplete ($counter/$files.count * 100)
 		Copy-Item $file.pspath $dest -Force
 		$counter++
 		}
 }
`

