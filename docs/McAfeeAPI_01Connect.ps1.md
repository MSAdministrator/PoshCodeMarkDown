---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3344
Published Date: 2012-04-11t14
Archived Date: 2012-04-14t02
---

# mcafeeapi_01connect - 

## Description

mcafee web api module functions. [set001]

## Comments



## Usage



## TODO



## function

`mcafee-connect`

## Code

`#
 #
 function McAfee-Connect{
 	param([String]$script:ServerURL="SERVERNAME:8443")
 	$c = McAfee-Credential
 	$script:wc = McAfee-WebClient -Credential $c
 }
 
 function McAfee-Credential{
 	$c = Get-Credential -Credential $null
 	return $c
 }
 
 function McAfee-WebClient{
 	param($Credential)
 	$wc = new-object system.net.webclient
 	$wc.credentials = New-Object System.Net.NetworkCredential -ArgumentList ($Credential.GetNetworkCredential().username,$Credential.GetNetworkCredential().password)
 	return $wc
 }
`

