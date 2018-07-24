---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5164
Published Date: 2014-05-17t23
Archived Date: 2014-05-20t22
---

# get-commanddefinition - 

## Description

this is more or less a port of forth’s “see” command to powershell.

## Comments



## Usage



## TODO



## module

`get-commanddefinition`

## Code

`#
 #
 New-Alias see Get-CommandDefinition
 
 #.SYNOPSIS
 #.LINK
 #.NOTES
 #
 function Get-CommandDefinition {
 	[OutputType([string])]
 	[CmdletBinding()]
 	param(
 		[Parameter(Mandatory = $true, Position = 0)]
 		[string[]]$Name
 ,
 		[Alias('PSSnapin')]
 		[string[]]$Module
 ,
 		[Alias('Type')]
 		[System.Management.Automation.CommandTypes]$CommandType
 	)
 
 	function qname {
 		if ($args[0].Module) { '{0}\{1}' -f $args[0].Module,$args[0].Name } else { $args[0].Name }
 	}
 
 	Get-Command @PSBoundParameters | Select-Object -First 1 |%{
 		$out = ''
 		$a = [System.Collections.Generic.HashSet[string]]([System.StringComparer]::OrdinalIgnoreCase)
 
 		if ($_ -is [System.Management.Automation.AliasInfo]) { $null = $a.Add($_.Name) }
 
 		while ($_ -is [System.Management.Automation.AliasInfo]) {
 			if ($a.Add($_.Definition)) {
 			} else {
 				break
 			}
 			if ($null -ceq $_.ReferencedCommand) { break }
 			$_ = $_.ReferencedCommand
 		}
 		if ($_ -is [System.Management.Automation.CmdletInfo]) {
 			if ($null -cne $_.ImplementingType) {
 			}
 		} elseif ($_ -is [System.Management.Automation.FunctionInfo]) {
 		}
 		if ($_ -is [System.Management.Automation.CommandInfo] -and $_ -isnot [System.Management.Automation.AliasInfo]) {
 			$out += $_.Definition
 		}
 		$out
 	}
 }
`

