---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1722
Published Date: 
Archived Date: 2010-03-31t09
---

# get-httpresponseuri - 

## Description

fetch the head for a url and return the responseuri. good for service-independent short-url lengthening ;)

## Comments



## Usage



## TODO



## function

`get-httpresponseuri`

## Code

`#
 #
 function Get-HttpResponseUri {
 #.Synopsis
 #.Description
 #.Parameter ShortUrl
    PARAM(
       [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$true)]
       [Alias("Uri","Url")]
       [string]$ShortUrl
    )
    $req = [System.Net.HttpWebRequest]::Create($ShortUrl)
    $req.Method = "HEAD"
    $response = $req.GetResponse()
    Write-Output $response.ResponseUri
 }
`

