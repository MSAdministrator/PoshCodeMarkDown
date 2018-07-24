---
Author: michaelvdnest
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3526
Published Date: 2012-07-18t02
Archived Date: 2012-07-20t07
---

# get-eastereastern - 

## Description

calculates the julian easter date in the julian calendar for a given year.

## Comments



## Usage



## TODO



## function

`get-eastereastern`

## Code

`#
 #
 function Get-EasterEastern {
 	Param(
 		[Parameter(Mandatory=$true)]
         [int] $Year
 	)
 	
 	$a = $Year % 4
 	
 	$b = $Year % 7
 	
 	$c = $Year % 19 	
 	
 	$d = ((19 * $c) + 15) % 30 	
 	
 	$e = ((2 * $a) + (4 * $b) + $e1 + 34) % 7 	
 	
 	$month = [Math]::Floor(($d + $e + 114) / 31)
 	
 	$day = (($d + $e + 114) % 31) + 1 	
 
 	$cal = New-Object -TypeName System.Globalization.JulianCalendar
 	
 	New-Object -TypeName System.DateTime -ArgumentList $Year,$month,$day,$cal
 	
 	<#
 		.SYNOPSIS
 			Calculates the Julian Easter date in the Julian calendar for a given year.
 
 		.DESCRIPTION
 			Calculates the Julian Easter date in the Julian calendar for a given year. This is not the Gregorian Easter now used by Western churches. Algorithm sourced from http://en.wikipedia.org/wiki/Computus, Meeus Julian algorithm.
 
 		.PARAMETER Year
 			The year to calculate easter for.
 
 		.EXAMPLE
 			PS C:\> Get-EasterEastern 2017
 
 		.INPUTS
 			System.Int32
 
 		.OUTPUTS
 			System.DateTime
 	#>
 }
`

