---
Author: patrick dreyer
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1379
Published Date: 2010-10-06t23
Archived Date: 2015-05-09t23
---

# exceptionhandling - 

## Description

facilitates exception handling as used to under c# or java.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 ##############################################################################################################
 ##############################################################################################################
 
 if (-not ($MyInvocation.line -match '^\. .')) {
 	Write-Error 'No Functions were loaded - you need to invoke with . scriptname'
 	return
 }
 
 #
 #
 #
 #
 #
 function try {
 	param (
 		[ScriptBlock] $command = $( throw "The parameter -Command is required." ),
 		[Object]      $catch,
 		[Object]      $catch1,
 		[Object]      $catch2,
 		[Object]      $catch3,
 		[Object]      $catch4,
 		[Object]      $catch5,
 		[Object]      $catch6,
 		[Object]      $catch7,
 		[Object]      $catch8,
 		[Object]      $catch9,
 		[ScriptBlock] $finally = {}
 	)
 	
 	$catches = @{}
 	($catch,$catch1,$catch2,$catch3,$catch4,$catch5,$catch6,$catch7,$catch8,$catch9) | ? { $_ -ne $null } | % {
 		if ($_ -is [System.Collections.Hashtable]) {
 			$catches += $_
 		} elseif ($_ -is [ScriptBlock]) {
 			if ($catches.Contains([System.Exception])) {
 				$catches.set_Item([System.Exception], $_)
 			} else {
 				$catches.Add([System.Exception], $_)
 			}
 		} else {
 			throw New-Object Exception("Unknown catch argument type'" + $_.GetType() + "'")
 		}
 	}
 	
 	& {
 		$local:ErrorActionPreference = "SilentlyContinue"
 		trap {
 			trap {
 				& {
 					trap { throw $_ }
 					&$finally
 				}
 				throw $_
 			}
 			$exType = $_.Exception.GetType()
 			$catch  = $null
 			do {
 				$catches.GetEnumerator() | ? { $exType -eq $_.Key } | % { $catch = $_.Value }
 				if ($catch -ne $null) {
 					break
 				}
 				$exType = $exType.BaseType
 			} while ($exType -ne $null)
 			if ($catch -eq $null) { throw $_ }
 			$_ | & { &$catch }
 		}
 		&$command
 	}
 	& {
 		trap { throw $_ }
 		&$finally
 	}
 }
`

