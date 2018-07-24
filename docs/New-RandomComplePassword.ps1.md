---
Author: naveen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6274
Published Date: 2016-03-31t04
Archived Date: 2016-11-18t01
---

# new-randomcomplepassword - 

## Description

generates a random password of a specified length.

## Comments



## Usage



## TODO



## function

`new-randomcomplexpassword`

## Code

`#
 #
 Function New-RandomComplexPassword ($length=20)
         $password = [System.Web.Security.Membership]::GeneratePassword($length,2)
         return $password
 New-RandomComplexPassword
`

