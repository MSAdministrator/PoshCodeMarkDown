---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 822
Published Date: 2009-01-23t15
Archived Date: 2016-03-05t09
---

# convert-powerpack2ps1 - 

## Description

converts powergui’s .powerpack files into ps1 script files – each script node, link and action get represented as a function with the element’s name and its code inside. you can then dot-source the file and use the functions in your scripts or command line. works both in powershell v1 and v2.

## Comments



## Usage



## TODO



## script

`output-link`

## Code

`#
 #
 #######################################################################
 ######################################################################
 ######################################################################
 #
 #
 #####################################################################
 param(
 	$PowerPackFile = $(throw 'Please supply  path to source powerpack file'),
 	$OutputFilePath = $(throw 'Please supply  path to output ps1 file')
 )
 
 
 function IterateTree {
 	param($segment)
 	if ( $segment.Type -like 'Script*' ) {
 		
 		$name = $segment.name -replace ' |\(|\)', ''
 		$code = $segment.script.PSBase.InnerText
 		
 @"
 
 ########################################################################
 ########################################################################
 function $name {
 $code
 }
 "@ | Out-File $OutputFilePath -Append		
 		
 	}
 	if ($segment.items.container -ne $null) {
 		$segment.items.container | ForEach-Object { IterateTree $_ }
 	}
 }
 
 
 function Output-Link {
 	PROCESS {
 		if ( $_.script -ne $null ) { 
 			$name = $_.name -replace ' |\(|\)', ''
 			$code = $_.script.PSBase.InnerText
 
 @"
 
 ########################################################################
 ########################################################################
 function $name {
 $code
 }
 "@ | Out-File $OutputFilePath -Append		
 		}
 	}
 }
 
 
 
 
 $sourcefile = Get-ChildItem $PowerPackFile
 if ($sourcefile -eq $null) { throw 'File not found' }
 	
 @"
 ########################################################################
 ########################################################################
 "@ | Out-File $OutputFilePath
 
 $pp = [XML] (Get-Content $sourcefile)
 
 @"
 
 "@ | Out-File $OutputFilePath -Append
 IterateTree $pp.configuration.items.container[0]
 
 @"
 
 "@ | Out-File $OutputFilePath -Append
 
 $pp.configuration.items.container[1].items.container | 
 	where { $_.id -eq '481eccc0-43f8-47b8-9660-f100dff38e14' } | ForEach-Object {
 		$_.items.item, $_.items.container | Output-Link
 	}
 
 
 @"
 
 "@ | Out-File $OutputFilePath -Append
 
 $pp.configuration.items.container[1].items.container | 
 	where { $_.id -eq '7826b2ed-8ae4-4ad0-bf29-1ff0a25e0ece' } | ForEach-Object {
 		$_.items.item, $_.items.container | Output-Link
 	}
`

