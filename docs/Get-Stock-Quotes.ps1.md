---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 752
Published Date: 2009-12-27t00
Archived Date: 2015-05-05t05
---

# get stock quotes - 

## Description

get-stockquotes gives a very easy way to get stock quotes using powershell ctp3’s new web services capabilities.

## Comments



## Usage



## TODO



## function

`get-stockquote`

## Code

`#
 #
 function Get-StockQuote {
 	param($symbols)
 
 	process {
 		$s = new-webserviceproxy -uri http://www.webservicex.net/stockquote.asmx
 
 		foreach ($symbol in $symbols) {
 			$result = [xml]$s.GetQuote($symbol)
 			$result.StockQuotes.Stock
 		}
 	}
 }
 
`

