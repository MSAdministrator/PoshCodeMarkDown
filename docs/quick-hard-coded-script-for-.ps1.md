---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5947
Published Date: 
Archived Date: 2015-07-31t04
---

#  - 

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

