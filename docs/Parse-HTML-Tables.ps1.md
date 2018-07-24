---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 561
Published Date: 2009-08-29t14
Archived Date: 2017-01-23t05
---

# parse html tables - 

## Description

a function to parse tables out of html files and return them as powershell objects.

## Comments



## Usage



## TODO



## function

`get-rowinner`

## Code

`#
 #
 #
 #
 
 function get-rowInner {
 	param($inputObject, $unique=0, $trim=0)
 
 	$values = @()
 	foreach ($obj in $inputObject) {
 		if ($obj.nodeName -eq "TD" -or $obj.nodeName -eq "TH") {
 			$value = $obj.IHTMLElement_innerText
 			if ($trim) {
 				$value = $value.trim()
 			}
 			if ($unique) {
 				if ($values -contains $value) {
 					$i = 2
 					while ($values -contains ($value + $i)) {
 						$i++
 					}
 					$values += ($value + $i)
 				} else {
 					$values += $value
 				}
 			} else {
 				$values += $value
 			}
 		}
 	}
 
 	if ($values.length -gt 0) {
 		return $values
 	} else {
 		return $null
 	}
 }	
 
 function get-row {
 	param($inputObject, $unique=0, $trim=0)
 
 	if ($inputObject.nodeName -eq "TR") {
 		return get-rowInner -inputObject $inputObject.childnodes -unique $unique -trim $trim
 	} else {
 		foreach ($node in $inputObject.childnodes) {
 			$row = get-row -inputObject $node -unique $unique -trim $trim
 			if ($row -ne $null) {
 				return $row
 			}
 		}
 	}
 }
 
 function get-table {
 	param($inputObjects)
 
 	$headings = $null
 	$rows = @()
 
 	foreach ($obj in $inputObjects) {
 		if ($headings -eq $null) {
 			$headings = get-row -inputObject $obj -unique 1 -trim 1
 			continue
 		}
 
 		$row = get-row -inputObject $obj
 		if ($row -ne $null -and $row.length -eq $headings.length) {
 			$rowObject = new-object psobject
 			for ($i = 0; $i -lt $headings.length; $i++) {
 				$value = $row[$i]
 				if ($value -eq $null) {
 					$value = ""
 				}
 				$rowObject | add-member -type noteproperty -name $headings[$i] -value $value
 			}
 			$rows += $rowObject
 		}
 	}
 
 	return $rows
 }
 
 function Parse-HtmlTableRecursive {
 	param($inputObjects)
 
 	foreach ($_ in $inputObjects) {
 		if ($_.nodeName -eq "TBODY") {
 			if (--$global:htmlParseCount -eq 0) {
 				return get-table -inputObjects $_.childnodes
 			}
 		}
 
 		if ($_.childnodes -ne $null) {
 			$table = Parse-HtmlTableRecursive -inputObjects $_.childnodes
 			if ($table) {
 				return $table
 			}
 		}
 	}
 
 	return $null
 }
 
 function Parse-HtmlTable {
 	param($url, $tableNumber=1)
 
 	$client = new-object net.webclient
 	$htmltext = $client.downloadstring($url)
 
 	#$temp = gc $url
 	#$htmltext = ''
 	#}
 
 	$global:htmlParseCount = $tableNumber
 	$h = new-object -com "HTMLFILE"
 	$h.IHTMLDocument2_write($htmltext)
 	$ret = Parse-HtmlTableRecursive -inputObject $h.body
 	remove-variable -scope global htmlParseCount
 	return $ret
 }
 
`

