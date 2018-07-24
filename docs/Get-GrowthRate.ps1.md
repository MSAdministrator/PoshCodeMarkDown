---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 704
Published Date: 
Archived Date: 2009-01-06t13
---

# get-growthrate - 

## Description

calculates percentage growth rate given a starting value, ending value, and number of periods in the range.  stahler thx!

## Comments



## Usage



## TODO



## function

`get-growthrate`

## Code

`#
 #
 function Get-GrowthRate {
 	param( $Start, $End, $Period ) 
 	$rate = [math]::Abs( [math]::Pow( ( $End / $Start ),( 1 / $Period - 1 ) ) - 1 )
 	"{0:P}" -f $rate
 }
`

