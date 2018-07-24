---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.01
Encoding: ascii
License: cc0
PoshCode ID: 854
Published Date: 
Archived Date: 2009-02-08t14
---

# help differ 10000 - 

## Description

help differ 10000

## Comments



## Usage



## TODO



## function

`get-helpxml`

## Code

`#
 #
 
 function Get-HelpXml {
 	param ($module)
 
 	get-command -module $module | % {
 		$allParams = $_.parametersets | % { $_.parameters | select -unique Name }
 		$obj = new-object psobject
 		$obj | add-member -type noteproperty -name name -value $_.Name
 		$obj | add-member -type noteproperty -name parameters -value $allParams
 
 		$h = $_ | get-help
 		$obj | add-member -type noteproperty -name description -value $h.Description
 		$obj | add-member -type noteproperty -name paramDescriptions -value $h.parameters
 
 		$obj
 	}
 }
 
 function Diff-HelpXml {
 	param ($beforeFile, $afterFile)
 
 	$before = import-clixml $beforeFile
 	$after = import-clixml $afterFile
 
 
 	echo "= New cmdlets in this release ="
 	echo " ||$titleColor Name ||$titleColor Description ||"
 	foreach ($a in $after) {
 		$match = $before | where { $_.Name -eq $a.Name }
 		if ($match -eq $null) {
 			$name = $a.Name
 			$desc = $a.Description[0].Text
 			echo " || $name || $desc ||"
 		}
 	}
 
 	echo ""
 	echo "= New cmdlet parameters in this release ="
 	$color = $color1
 	echo " ||$titleColor Cmdlet Name ||$titleColor Parameter Name ||$titleColor Description ||"
 	foreach ($a in $after) {
 		$match = $before | where { $_.Name -eq $a.Name }
 		if ($match) {
 			$noMatches = $true
 			foreach ($p in $a.parameters) {
 				$matchPar = $match.parameters | where { $_.Name -eq $p.Name }
 				if ($matchPar -eq $null) {
 					$cmdletName = ""
 					$pName = $p.Name
 					if ($noMatches) {
 						$cmdletName = $a.Name
 						$noMatches = $false
 						if ($color -eq $color1) {
 							$color = $color2
 						} else {
 							$color = $color1
 						}
 					}
 					$matchDesc = $a.paramDescriptions.parameter | where { $_.Name -eq $pName }
 					$pDesc = $matchDesc.description[0].text
 					$pDesc = $pDesc -replace "[^A-Za-z1-9\.`"``, ]", " "
 
 					echo " ||$color $cmdletName ||$color $pName ||$color $pDesc ||"
 				}
 			}
 		}
 	}
 }
 
`

