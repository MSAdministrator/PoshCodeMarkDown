---
Author: munsonisim
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5906
Published Date: 2015-06-24t15
Archived Date: 2015-06-26t07
---

# global zip functions - 

## Description

extract-zip got me thinking – i’ve had some global zip functions in my .profile for a long ass time – here they are, i suggest if you want to allow “global” use of these, to add them to the global profile for powershell. i’m sure i copied them form some ps guru, but the whom escapes me…

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function global:New-Zip
 {
 	param([string]$zipfilename)
 	set-content $zipfilename ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))
 	(dir $zipfilename).IsReadOnly = $false
 }
 function global:Add-Zip
 {
 	param([string]$zipfilename)
 
 	if(-not (test-path($zipfilename)))
 	{
 		set-content $zipfilename ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))
 		(dir $zipfilename).IsReadOnly = $false	
 	}
 	
 	$shellApplication = new-object -com shell.application
 	$zipPackage = $shellApplication.NameSpace($zipfilename)
 	
 	foreach($file in $input) 
 	{ 
             $zipPackage.CopyHere($file.FullName)
             Start-sleep -milliseconds 500
 	}
 }
 function global:Get-Zip
 {
 	param([string]$zipfilename)
 	if(test-path($zipfilename))
 	{
 		$shellApplication = new-object -com shell.application
 		$zipPackage = $shellApplication.NameSpace($zipfilename)
 		$zipPackage.Items() | Select Path
 	}
 }
 function global:Extract-Zip
 {
 	param([string]$zipfilename, [string] $destination)
 
 	if(test-path($zipfilename))
 	{	
 		$shellApplication = new-object -com shell.application
 		$zipPackage = $shellApplication.NameSpace($zipfilename)
 		$destinationFolder = $shellApplication.NameSpace($destination)
 		$destinationFolder.CopyHere($zipPackage.Items())
 	}
 }
`

