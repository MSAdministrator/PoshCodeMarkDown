---
Author: tst-wms-print-00
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6211
Published Date: 2016-02-10t20
Archived Date: 2016-04-25t18
---

# get-hostname - 

## Description

print the hostname of the system. complete with v2 comment-based help, but works fine on v1.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 param (
 	[switch]$Short		= $true,
 	[switch]$Domain		= $false,
 	[switch]$FQDN		= $false
 )
 
 $ipProperties = [System.Net.NetworkInformation.IPGlobalProperties]::GetIPGlobalProperties()
 if ( $FQDN ) {
 	return "{0}.{1}" -f $ipProperties.HostName, $ipProperties.DomainName
 }
 if ( $Domain ) {
 	return $ipProperties.DomainName
 }
 if ( $Short ) {
 	return $ipProperties.HostName
 }
`

