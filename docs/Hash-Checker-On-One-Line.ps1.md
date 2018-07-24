---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 988
Published Date: 2009-04-01t11
Archived Date: 2016-05-17t14
---

# hash checker on one line - 

## Description

check and md5 or sha1 hash in a “single line” of powershell.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 &{ 
   PARAM($FileName,$HashFileName) 
   ((Get-Content $HashFileName) -match $FileName)[0].split(" ")[0] -eq 
   [string]::Join("", (
     [Security.Cryptography.HashAlgorithm]::Create(
       ([IO.Path]::GetExtension($HashFileName).Substring(1).ToUpper())
     ).ComputeHash( 
       [IO.File]::ReadAllBytes( (Convert-Path $FileName) 
     )
   ) | ForEach { "{0:x2}" -f $_ }))
 } npp.5.3.1.Installer.exe npp.5.3.1.release.md5
`

