---
Author: rich kusak
Publisher: 
Copyright: 
Email: 
Version: 1.0.0.0
Encoding: ascii
License: cc0
PoshCode ID: 2661
Published Date: 2012-05-06t12
Archived Date: 2012-12-29t04
---

# get-mypublicipaddress - 

## Description

the get-mypublicipaddress script uses dns-o-matic, an opendns resource, to retreive the public ip address that represents your computer on the internet.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 	.SYNOPSIS
 		Gets the public IP address that represents your computer on the internet.
 	
 	.DESCRIPTION
 		The Get-MyPublicIPAddress script uses DNS-O-Matic, an OpenDNS resource, to retreive the public IP address that represents your computer on the internet.
 	
 	.EXAMPLE
 		Get-MyPublicIPAddress
 		1.1.1.1
 		
 		Description
 		-----------
 		This command returns the public IP address that represents your computer on the internet.
 	
 	.INPUTS
 		None
 	
 	.OUTPUTS
 		System.String
 	
 	.NOTES
 		Name: Get-MyPublicIPAddress
 		Author: Rich Kusak
 		Created: 2010-08-30
 		LastEdit: 2010-08-30 11:05
 		Version: 1.0.0.0
 		
 	.LINK
 		http://myip.dnsomatic.com
 
 	.LINK
 		http://www.opendns.com	
 #>
 
 	[CmdletBinding()]
 	param ()
 	
 	$webClient = New-Object System.Net.WebClient
 	
 	$webClient.DownloadString('http://myip.dnsomatic.com/')
`

