---
Author: crazydave
Publisher: 
Copyright: 
Email: 
Version: 2.6
Encoding: ascii
License: cc0
PoshCode ID: 643
Published Date: 2009-10-19t05
Archived Date: 2016-05-23t02
---

# lookup-wirelesslocation - 

## Description

this simple function translates a wireless ap’s bbid (mac) through the skyhookwireless api’s to return location information…

## Comments



## Usage



## TODO



## function

`lookup-location`

## Code

`#
 #
 function LookUp-Location {
 param([String] $mac)
 $mac = $mac.Replace(":","").Replace("-","")
 
 $wClient = New-Object System.Net.WebClient
 $body = "<?xml version='1.0'?><LocationRQ xmlns='http://skyhookwireless.com/wps/2005' version='2.6' street-address-lookup='full'><authentication version='2.0'><simple><username>beta</username><realm>js.loki.com</realm></simple></authentication><access-point><mac>#{mac}</mac><signal-strength>-50</signal-strength></access-point></LocationRQ>"
 $url = "https://api.skyhookwireless.com/wps2/location"
 $wClient.Headers.Add("Content-Type:text/xml")
 $body = $body.Replace("#{mac}", $mac)
 
 $response = $wClient.UploadString($url, $body)
 return $response
 }
`

