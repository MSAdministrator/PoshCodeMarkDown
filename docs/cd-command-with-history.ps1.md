---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 697
Published Date: 
Archived Date: 2009-01-06t13
---

# cd command with history - 

## Description

i got this script originally from jim truher and have tweaked it a bit over time (nothing major). basically it replaces your cd function with one that keeps a history.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ########################################################
 #
 #
 if( test-path alias:\cd ) { remove-item alias:\cd }
 $global:PWD = get-location;
 $global:CDHIST = [System.Collections.Arraylist]::Repeat($PWD, 1);
 function cd {
 	$cwd = get-location;
 	$l = $global:CDHIST.count;
 
 	if ($args.length -eq 0) { 
 		set-location $HOME;
 		$global:PWD = get-location;
 		$global:CDHIST.Remove($global:PWD);
 		if ($global:CDHIST[0] -ne $global:PWD) {
 			$global:CDHIST.Insert(0,$global:PWD);
 		}
 		$global:PWD;
 	}
 	elseif ($args[0] -like "-[0-9]*") {
 		$num = $args[0].Replace("-","");
 		$global:PWD = $global:CDHIST[$num];
 		set-location $global:PWD;
 		$global:CDHIST.RemoveAt($num);
 		$global:CDHIST.Insert(0,$global:PWD);
 		$global:PWD;
 	}
 	elseif ($args[0] -eq "-l") {
 		for ($i = $l-1; $i -ge 0 ; $i--) { 
 			"{0,6}  {1}" -f $i, $global:CDHIST[$i];
 		}
 	}
 	elseif ($args[0] -eq "-") { 
 		if ($global:CDHIST.count -gt 1) {
 			$t = $CDHIST[0];
 			$CDHIST[0] = $CDHIST[1];
 			$CDHIST[1] = $t;
 			set-location $global:CDHIST[0];
 			$global:PWD = get-location;
 		}
 		$global:PWD;
 	}
 	else { 
 		set-location "$args";
 	$global:PWD = pwd; 
 		for ($i = ($l - 1); $i -ge 0; $i--) { 
 			if ($global:PWD -eq $CDHIST[$i]) {
 				$global:CDHIST.RemoveAt($i);
 			}
 		}
 
 		$global:CDHIST.Insert(0,$global:PWD);
 		$global:PWD;
 	}
 
 	$global:PWD = get-location;
 }
`

