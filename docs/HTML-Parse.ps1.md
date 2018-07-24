---
Author: daniel srlv
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6337
Published Date: 2016-05-10t04
Archived Date: 2016-05-19t19
---

# html parse - 

## Description

this is a very meaningless demo showing how to get and work with a html document in parsed form. the actual script gets a html-table from www.apk.se (listing cheap alcohol in sweden) and parses and converts that to a object and hands that of to the pipe.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $page = Invoke-WebRequest "http://www.apk.se"
 $html = $page.parsedHTML
 $products = $html.body.getElementsByTagName("TR")
 $headers = @()
 foreach($product in $products)
 {
 	$colID = 0;
 	$hRow = $false
 	$returnObject = New-Object Object
 	foreach($child in $product.children)
 	{	
 		if ($child.tagName -eq "TH")
 		{
 			$headers += @($child.outerText)
 			$hRow = $true
 		}
 
 		if ($child.tagName -eq "TD")
 		{
 			$returnObject | Add-Member NoteProperty $headers[$colID] $child.outerText
 		}
 		$colID++
 
 	}
 	if (-not $hRow) { $returnObject }
 }
`

