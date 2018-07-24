---
Author: michaelvdnest
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3527
Published Date: 2012-07-18t02
Archived Date: 2012-07-20t07
---

# get-easterwestern - 

## Description

calculates the gregorian easter date for a given year.

## Comments



## Usage



## TODO



## function

`get-easterwestern`

## Code

`#
 #
 function Get-EasterWestern {
 	Param(
 		[Parameter(Mandatory=$true)]
         [int] $Year
  	)
 
 	
 	$a = $Year % 19
 
 	$b = [Math]::Floor($Year / 100)
 
 	$c = $Year % 100
 
 	$d = [Math]::Floor($b / 4)
 
 	$e = $b % 4
 
 	$f = [Math]::Floor(($b + 8) / 25)
 
 	$g = [Math]::Floor((($b - $f + 1) / 3))
 	
 	$h = ((19 * $a) + $b + (-$d) + (-$g) + 15) % 30
 
 	$i = [Math]::Floor($c / 4)
 
 	$k = $c % 4
 
 	$L = (32 + (2 * $e) + (2 * $i) + $L1) % 7
 
 	$m = [Math]::Floor(($a + (11 * $h) + (22 * $L)) / 451)
 
 	$month = [Math]::Floor(($h + $L + $v1 + 114) / 31)
 
 	$day = (($h + $L + $v1 + 114) % 31) + 1
 
 	[System.DateTime]$date = "$Year/$month/$day"
 	
 	$date
 	
 	<#
 		.SYNOPSIS
 			Calculates the Gregorian Easter date for a given year.
 
 		.DESCRIPTION
 			Calculates the Gregorian Easter date for a given year greater than 1751. Algorithm sourced from http://en.wikipedia.org/wiki/Computus, Anonymous Gregorian algorithm. 
 			
 		.PARAMETER Year
 			The year to calculate easter for.
 
 		.EXAMPLE
 			PS C:\> Get-EasterWestern 2017
 
 		.INPUTS
 			System.Int32
 
 		.OUTPUTS
 			System.DateTime
 	#>
 	
 	
 }
`

