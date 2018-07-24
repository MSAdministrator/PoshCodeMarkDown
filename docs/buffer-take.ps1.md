---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5132
Published Date: 2014-04-30t20
Archived Date: 2014-05-05t02
---

# buffer/take - 

## Description

port of perl 6’s gather/take but for appending to a string instead of to a list.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function buffer {
 #.SYNOPSIS
 #.PARAMETER Using
 #.EXAMPLE
 #.NOTES
 #.LINK
 	[CmdletBinding(DefaultParameterSetName = 'T')]
 	param(
 		[Parameter(Mandatory, Position = 0)]
 		[scriptblock]$Script
 ,
 		[Parameter(ParameterSetName='S')]
 		[string]$Separator
 ,
 		[Parameter(ParameterSetName='T')]
 		[string]$Terminator
 ,
 		[ValidateNotNull()]
 		[System.Text.StringBuilder]$Using
 	)
 	try {
 		${function:take} = (New-Object System.Management.Automation.PSModuleInfo $true).NewBoundScriptBlock({
 			if (0 -clt $script:state) { $script:acc.Append($script:sep) }
 			foreach ($null in $args) { $script:acc.Append([string]$foreach.Current) }
 			if (0 -ceq $script:state) { $script:acc.Append($script:sep) } else { $script:state = 1 }
 		})
 		& ${function:take}.Module {
 			$script:acc, $script:sep, $script:state = $args
 		}	$(if ($PSBoundParameters.ContainsKey('Using')) { $Using } else { [System.Text.StringBuilder]0 }) `
 			$(if ($PSCmdlet.ParameterSetName -ceq 'S') { $Separator } else { $Terminator }) `
 			(-($PSCmdlet.ParameterSetName -ceq 'S'))
 	} catch {
 		throw
 	}
 	$null = & $Script
 	if (-not $PSBoundParameters.ContainsKey('Using')) {
 		& ${function:take}.Module { $script:acc.ToString() }
 	}
 }
`

