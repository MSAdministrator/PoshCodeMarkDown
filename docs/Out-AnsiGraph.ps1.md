---
Author: xcud
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1335
Published Date: 
Archived Date: 2009-09-28t13
---

# out-ansigraph - 

## Description

pipe numeric output into a horizontal ansi barchart in psh. poshcode doesn’t handle ansi characters. replace &#9608; with an ansi block character. copy/paste from http

## Comments



## Usage



## TODO



## function

`out-ansigraph`

## Code

`#
 #
 #
 #
 function Out-AnsiGraph($Parameter1=$null) {
 	BEGIN {
 		$q = new-object Collections.queue
 	}
 
 	PROCESS {
 		if($_) {
 			$name = $_.($Parameter1[0]);
 			$val = $_.($Parameter1[1])
 			if($max -lt $val) { $max = $val}		 
 			if($namewidth -lt $name.length) { $namewidth = $name.length }
 			$q.enqueue(@($name, $val))			
 		}
 	}
 
 	END {
 		$q | %{
 			$name = "{0,$namewidth}" -f $_[0]
 			"$name $graph " + $_[1]
 		}
 
 	}
 }
 
 Export-ModuleMember Out-AnsiGraph
`

