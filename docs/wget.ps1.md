---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 120
Published Date: 2008-01-22t09
Archived Date: 2017-04-30t10
---

# wget - 

## Description

the simplest form of wget … will become get-fromweb or something …

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param([string]$url, [string]$path)
 
 if(!(Split-Path -parent $path) -or !(Test-Path -pathType Container (Split-Path -parent $path))) {
   $path = Join-Path $pwd (Split-Path -leaf $path)
 }
 
 "Downloading [$url]`nSaving at [$path]"
 $client = new-object System.Net.WebClient
 $client.DownloadFile( $url, $path )
 
 $path
`

