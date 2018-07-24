---
Author: jiayuhui
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5912
Published Date: 2015-06-29t03
Archived Date: 2015-07-03t01
---

# ipaupload - 

## Description

quick hard coded script for uploading a file to an ftp server.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $File = "D:\Dev\somefilename.zip"
 $ftp = "ftp://username:password@example.com/pub/incoming/somefilename.zip"
 
 "ftp url: $ftp"
 
 $webclient = New-Object System.Net.WebClient
 $uri = New-Object System.Uri($ftp)
 
 "Uploading $File..."
 
 $webclient.UploadFile($uri, $File)
`

