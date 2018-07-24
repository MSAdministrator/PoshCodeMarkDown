---
Author: steven murawski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 968
Published Date: 2011-03-20t05
Archived Date: 2011-05-04t04
---

# get-staticmethoddefin - 

## Description

helper function to list the definitions of static methods

## Comments



## Usage



## TODO



## function

`get-staticmethoddefinition`

## Code

`#
 #
 
 
 function Get-StaticMethodDefinition()
 {
 	param ([string[]]$Method, [Type]$Type=$null)
 	BEGIN
 	{
 		if ($Type -ne $null)
 		{
 			$Type | Get-StaticMethodDefinition $Method
 		}
 	}
 	
 	PROCESS
 	{
 		if ($_ -ne $null)
 		{
 			$_ | Get-Member -Name $Method -Static -MemberType Method | ForEach-Object {$_.Definition -replace '\), ', "), `n"}
 		}
 	}
 }
`

