---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5934
Published Date: 2016-07-14t18
Archived Date: 2016-06-10t15
---

# ftp listdirectory - 

## Description

an example showing how to get a file listing via ftp.  note

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $cred = Get-Credential
 
 [System.Net.FtpWebRequest]$request = [System.Net.WebRequest]::Create("ftp://joelbennett.net")
 $request.Credentials = $cred
 
 $response = $request.GetResponse()
 
 $list = Receive-Stream $response.GetResponseStream()
`

