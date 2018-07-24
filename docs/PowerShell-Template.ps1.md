---
Author: gene magerr
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 778
Published Date: 
Archived Date: 2010-01-26t13
---

# powershell template - 

## Description

i’ve modified the original function. i like this one better.

## Comments



## Usage



## TODO



## function

`new-script`

## Code

`#
 #
 Function New-Script
 {
 $strName = $env:username
 $date = get-date -format d
 $name = Read-Host "Filename"
 if ($name -eq "") { $name="NewTemplate" }
 $email = Read-Host "eMail Address"
 if ($email -eq "") { $email="youremail@yourhost.com" }
 $file = New-Item -type file "$name.ps1" -force
 $template=@"
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 "@ | out-file $file
 ii $file
 }
`

