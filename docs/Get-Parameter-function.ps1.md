---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1053
Published Date: 2010-04-22t12
Archived Date: 2016-03-05t10
---

# get-parameter function - 

## Description

get a dictionary of parameters for a function or cmdlet, optionally including the common parameters (verbose, debug etc) for functions using cmdletbinding, or ordinary cmdlets.

## Comments



## Usage



## TODO



## function

`get-parameters`

## Code

`#
 #
 
 function Get-Parameters {
 	param([string]$CommandName, [switch]$IncludeCommon)
 	
 	try {
 		$command = get-command $commandname
 		$parameters = (new-object System.Management.Automation.CommandMetaData $command, $includecommon).Parameters
          } catch {
 		write-warning "Could not find command or obtain parameters for $commandname"
          }
 }
`

