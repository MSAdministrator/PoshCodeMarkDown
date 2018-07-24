---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1732
Published Date: 
Archived Date: 2010-04-09t14
---

# get-factualdata - 

## Description

retrieve data from factual.com. note

## Comments



## Usage



## TODO



## function

`get-factualdata`

## Code

`#
 #
 #
 Function Get-FactualData {
 	param($tableName)
 
 	$wc = New-Object Net.Webclient
 	$url = "http://api.factual.com/v1/" + $APIKEY + "/tables/" + $tableName + "/read.xml"
 	$xml = [xml]$wc.DownloadString($url)
 
 	$nFields = $xml.result.response.fields.field.length - 1
 	$fields = $xml.result.response.fields.field[1 .. ($nFields + 1)]
 
 	$xml.result.response.data.row | Foreach {
 		$t = "" | Select $fields
 		foreach ($i in 1 .. $nFields) {
 			$t.($fields[$i-1]) = $_.cell[$i]
 		}
 		Write-Output $t
 	}
 }
`

