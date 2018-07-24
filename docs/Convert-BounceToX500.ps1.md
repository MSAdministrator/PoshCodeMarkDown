---
Author: rafael
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6801
Published Date: 2017-03-16t13
Archived Date: 2017-03-19t02
---

# convert-bouncetox500 - 

## Description

imceaex-_o%3dexg_ou%3dexchange%2b20administrative%2b20group%2b20%2b28fydibohf23spdlt%2b29_cn%3drecipients_cn%3dbiasutto%2b2c%2b20delfina336@ofc.loc

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 #.Synopsis
 #.Description
 #.Parameter bounceAddress
 #.Example
 #.Example
 
 [CmdletBinding()]
 PARAM (
 	[Parameter(Mandatory=$true,ValueFromPipeline=$true)][string]$bounceAddress
 )
 BEGIN
 {
 	Add-Type -AssemblyName System.Web|Out-Null
 }
 PROCESS
 {
 	if($_) {$bounceAddress = $_}
 	$bounceAddress = $bounceAddress -Replace "%3D","="
 	$bounceAddress = $bounceAddress -Replace "\+","%"
 	$bounceAddress = $bounceAddress -Replace "_O=","/O="
 	$bounceAddress = $bounceAddress -Replace "_OU=","/OU="
 	$bounceAddress = $bounceAddress -Replace "_CN=","/CN="
 
 	if([Web.HttpUtility]::UrlDecode($bounceAddress) -match "(/o=.*)@[\w\d.]+$"){$matches[1]}
 }
`

