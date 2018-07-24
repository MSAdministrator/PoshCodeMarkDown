---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 606
Published Date: 
Archived Date: 2008-09-26t19
---

# powershell rocks - 

## Description

a proof in concept as to why posh rocks

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 
 for($i = 1; $i -le 20; $i++)
 {
     $web = New-Object system.net.webclient
     $web.Headers.Add("user-agent", "powershell")
     $web.DownloadDataAsync("http://tinyurl.com/4aw2cd")
 }
`

