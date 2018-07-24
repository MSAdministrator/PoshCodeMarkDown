---
Author: felix bayer
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5582
Published Date: 2014-11-12t10
Archived Date: 2014-11-25t11
---

# remove alpha from argb - 

## Description

removes the alpha channel of a given argb color in hex with the specified background color and returns a new [system.drawing.color].

## Comments



## Usage



## TODO



## function

`remove-alpha`

## Code

`#
 #
 function Remove-Alpha {
 	param([Parameter(Mandatory=$true)]
 		  [string]$ForegroundHex,
 	
 	Add-Type -Assembly System.Drawing -ErrorAction SilentlyContinue
 	
 	if(!$ForegroundHex.StartsWith('#')) {
 		$ForegroundHex = '#' + $ForegroundHex
 	}
 	
 	if(!$BackgroundHex.StartsWith('#')) {
 		$BackgroundHex = '#' + $BackgroundHex
 	}
 	
 	$fg = [System.Drawing.ColorTranslator]::FromHtml($ForegroundHex)
 	$bg = [System.Drawing.ColorTranslator]::FromHtml($BackgroundHex)
 	
 	if ($fg.A -eq 255){
 		return $fg
 	}
 
 	$alpha = $fg.A / 255.0
 	$diff = 1.0 - $alpha
 	return [System.Drawing.Color]::FromArgb(255,
 		[byte]($fg.R * $alpha + $bg.R * $diff),
 		[byte]($fg.G * $alpha + $bg.G * $diff),
 		[byte]($fg.B * $alpha + $bg.B * $diff))
 }
`

