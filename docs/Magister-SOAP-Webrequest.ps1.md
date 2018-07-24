---
Author: paul wiegmans
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5791
Published Date: 2016-03-19t08
Archived Date: 2016-03-05t14
---

# magister soap webrequest - 

## Description

this powershell code performs a soap webrequest to the school

## Comments



## Usage



## TODO



## function

`doe-webquery`

## Code

`#
 #
 Clear
 Set-StrictMode -Version 2
 
 $mijnpad = Split-Path -parent $MyInvocation.MyCommand.Definition
 Echo "Mijn pad is: $mijnpad "
 $layout = "Basis"
 $username = "user"
 $passwd = "pass"
 $WebqueryUrl = "https://bonhoeffer.swp.nl:8800/doc?Function=GetData"`
 	+"&Library=Data&SessionToken=$username%3B$passwd&Layout=$layout"`
 	+"&Parameters=leerjaar%3D1112&Type=CSV"
 
 function Doe-Webquery ($Url) {
 	
 	$request = [System.Net.WebRequest]::Create($Url)
 	$response = $request.GetResponse()
 	$requestStream = $response.GetResponseStream()
 	$MagisterEncoding = [System.Text.Encoding]::UTF7  
 	$readStream = new-object System.IO.StreamReader( $requestStream, $MagisterEncoding )
 
 
 	$readStream.Close()
 	$response.Close()
 	return $result
 }
 
 $lines = Doe-Webquery $WebQueryUrl
 Echo "Webquery geeft $($lines.count) regels"
 
`

