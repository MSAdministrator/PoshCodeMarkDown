---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6094
Published Date: 
Archived Date: 2016-03-18t23
---

#  - 

## Description

invoke-restmethod example

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $headers = @{
 	'Content-Type' = "application/x-www-form-urlencoded"
 	'Accept' = "application/json"
 	}
 
 $response = Invoke-RestMethod -Uri $uri -Method POST -Header $headers -Credentail (Get-Credential)
`

