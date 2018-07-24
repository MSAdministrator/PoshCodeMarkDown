---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3357
Published Date: 2012-04-16t02
Archived Date: 2016-11-28t14
---

# test-webdav - 

## Description

quickly tests if a given web server (specified by url parameter) is running a webdav service.  should work against any server platform that supports the webdav rfcs.

## Comments



## Usage



## TODO



## function

`test-webdav`

## Code

`#
 #
 function Test-WebDav ()
 {
 	param ( $Url = "$( throw 'URL parameter is required.')" )
 	$xhttp = New-Object -ComObject msxml2.xmlhttp
 	$xhttp.open("OPTIONS", $url, $false)
 	$xhttp.send()
 	if ( $xhttp.getResponseHeader("DAV") ) { $true }
 	else { $false }
 }
`

