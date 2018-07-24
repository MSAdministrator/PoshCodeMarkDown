---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 2.0.1
Encoding: ascii
License: cc0
PoshCode ID: 1407
Published Date: 
Archived Date: 2009-10-26t15
---

# get-shorturl - 

## Description

uses the bit.ly api to create a short url. requires a login and api key from http

## Comments



## Usage



## TODO



## function

`get-shorturl`

## Code

`#
 #
 Function Get-ShortURL {
 	Param($longURL, $login, $apiKey)	
 	$url = "http://api.bit.ly/shorten?version=2.0.1&format=xml&longUrl=$longURL&login=$login&apiKey=$apikey"
 	$request = [net.webrequest]::Create($url)
 	$responseStream = new-object System.IO.StreamReader($request.GetResponse().GetResponseStream())
 	$response = $responseStream.ReadToEnd()
 	$responseStream.Close()
 	
 	$result = [xml]$response
 	Write-Output $result.bitly.results.nodeKeyVal.shortUrl
 }
`

