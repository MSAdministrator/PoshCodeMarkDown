---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 849
Published Date: 
Archived Date: 2009-02-08t14
---

# convertto-function - 

## Description

this script takes a path to a script (full or relative), a fileinfo object, or either as pipeline input.  it converts the script’s content to a function of the same name as the file.  for example, ./convertto-function get-server.ps1 would create a function called get-server.  if the function already exists, it will replace it with the new script.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 param ($filename)
 
 PROCESS
 {
 	if ($_ -ne $Null)
 	{
 		$filename = $_
 	}
 	
 	if ($filename -is [System.IO.FileInfo])
 	{
 		$filename = $filename.Name
 	}
 	
 	if (Test-Path $filename) 
 	{	
 		
 		$name = (Resolve-Path $filename | Split-Path -Leaf) -replace '\.ps1'	
 		
 		$scriptblock = get-content $filename | Out-String
 		
 		if (Test-Path function:global:$name)
 		{
 			Set-Item -Path function:global:$name -Value $scriptblock 
 			Get-Item -Path function:global:$name
 		}
 		else
 		{
 			New-Item -Path function:global:$name -Value $scriptblock
 		}
 	}
 	else 
 	{
 		throw 'Either a valid path or a FileInfo object'
 	}
 }
`

