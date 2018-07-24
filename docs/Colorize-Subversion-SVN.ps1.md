---
Author: jim palmer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 894
Published Date: 
Archived Date: 2009-02-25t08
---

# colorize subversion svn - 

## Description

colorize subversion svn stat output.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function ss () {
 	$c = @{ "A"="Magenta"; "D"="Red"; "C"="Yellow"; "G"="Blue"; "M"="Cyan"; "U"="Green"; "?"="DarkGray"; "!"="DarkRed" }
 	foreach ( $svno in svn stat ) {  
 		$color = $c[$svno.SubString(0,1).ToUpper()]
 		if ( $color ) { 
 			write-host $svno -Fore $color
 		} else { 
 			write-host $svno
 		}
 	}
 }
`

