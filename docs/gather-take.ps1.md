---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 5131
Published Date: 2014-04-30t20
Archived Date: 2014-05-05t01
---

# gather/take - 

## Description

port of perl 6’s gather/take (which itself is a port of mathematica’s reap/sow)

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function gather {
 #.SYNOPSIS
 #.PARAMETER Using
 #.EXAMPLE
 #.NOTES
 	[CmdletBinding()]
 	param(
 		[Parameter(Mandatory)]
 		[scriptblock]$Script
 ,
 		[ValidateNotNull()]
 		[System.Collections.IList]$Using
 	)
 	try {
 		${function:take} = (New-Object System.Management.Automation.PSModuleInfo $true).NewBoundScriptBlock({
 			foreach ($null in $args) { $null = $script:acc.Add([string]$foreach.Current) }
 		})
 		& ${function:take}.Module { $script:acc = $args[0] } $(if ($PSBoundParameters.ContainsKey('Using')) { ,$Using } else { ,[System.Collections.ArrayList]@() })
 	} catch {
 		throw
 	}
 	$null = & $Script
 	if (-not $PSBoundParameters.ContainsKey('Using')) {
 		& ${function:take}.Module { $script:acc }
 	}
 }
`

