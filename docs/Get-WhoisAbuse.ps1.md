---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4793
Published Date: 
Archived Date: 2015-03-23t19
---

#  - 

## Description

a function to return the abuse email address from arin.net.

## Comments



## Usage



## TODO



## function

`get-whoisabuse`

## Code

`#
 #
 function get-whoisabuse ([string]$ipaddress)
 {
 
 [xml]$a = (Invoke-WebRequest -Uri "http://whois.arin.net/rest/ip/$ip" -ContentType "text/xml").content
 
 [xml]$pocs = (Invoke-WebRequest -Uri ("http://whois.arin.net/rest/net/" + $a.net.handle + "/pocs") -ContentType "text/xml").content
 
 
 [array]$result = $abuse.poc.emails.email
 
 $result
 
 }
`

