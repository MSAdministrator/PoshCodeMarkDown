---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6204
Published Date: 2016-02-07t19
Archived Date: 2016-03-18t22
---

# quoted printable - 

## Description

quoted printable decoder (assumes you want to output strings as opposed to a byte buffer).

## Comments



## Usage



## TODO



## function

`convertfrom-quotedprintable`

## Code

`#
 #
 function ConvertFrom-QuotedPrintable {
 	[OutputType([string])]
 	[CmdletBinding()]
 	param(
 		[Parameter(ValueFromPipeline)]
 		[string]$InputObject
 ,
 		[Parameter(Mandatory)]
 		[System.Text.Encoding]$Encoding = [System.Text.Encoding]::UTF8
 	)
 	begin {
 		$buf = ''
 	}
 	process {
 		$buf += [regex]::Replace(
 			[regex]::Replace($InputObject, "=[ `t]*(?:`r`n|`n|\z)", ''),
 			'=[0-9a-fA-F]{2}',
 			[System.Text.RegularExpressions.MatchEvaluator]{[string][char][int]::Parse($args[0].Value.Substring(1), [System.Globalization.NumberStyles]::AllowHexSpecifier)}
 		)
 	}
 	end {
 		$Encoding.GetString([byte[]][char[]]$buf)
 	}
 }
`

