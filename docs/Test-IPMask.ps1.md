---
Author: rich kusak
Publisher: 
Copyright: 
Email: 
Version: 1.0.0
Encoding: ascii
License: cc0
PoshCode ID: 2439
Published Date: 2011-01-04t22
Archived Date: 2011-04-24t10
---

# test-ipmask - 

## Description

the .net framework has the system.net.ipaddress class which can be used to validate a string as an ip address. i wanted to do the same with ip masks as well and came up with this script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 	.SYNOPSIS
 		Tests for a valid IP mask.
 	
 	.DESCRIPTION
 		The Test-IPMask script validates the input string against all CIDR subnet masks and returns a boolean value.
 	
 	.PARAMETER IPMask
 		The IP mask to be evaluated.
 	
 	.EXAMPLE
 		Test-IPMask 255.255.255.0
 		Description
 		-----------
 		Tests if the IP mask "255.255.255.0" is a valid CIDR subnet mask.
 	
 	.INPUTS
 		System.String
 		
 	.OUTPUTS
 		System.String
 		System.Boolean
 	
 	.NOTES
 		Name: Test-IPMask.ps1
 		Author: Rich Kusak (rkusak@cbcag.edu)
 		Created: 2011-01-03
 		LastEdit: 2011-01-05 00:49
 		Version 1.0.0
 		
 
 	.LINK
 		http://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
 
 	.LINK
 		about_regular_expressions
 	
 #>
 
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
 		[string]$IPMask
 	)
 
 	begin {
 
 		$common = '0|128|192|224|240|248|252|254|255'
 		
 		$masks = @(
 			"(^($common)\.0\.0\.0$)"
 			"(^255\.($common)\.0\.0$)"
 			"(^255\.255\.($common)\.0$)"
 			"(^255\.255\.255\.($common)$)"
 		)
 		
 		$regex = [string]::Join('|', $masks)
 	}
 
 	process {
 
 		[regex]::Match($IPMask, $regex) | Select @{name='IPMask' ; expression={$IPMask}},
 			@{name='Valid' ; expression={$_.Success}}
 	}
`

