---
Author: sddrcerrr
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2698
Published Date: 2012-05-27t11
Archived Date: 2016-03-19t00
---

# verifycategoryrule.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Set-StrictMode -Version Latest
 
 if($message.Body -match "book")
 {
     [Console]::WriteLine("This is a message about the book.")
 }
 else
 {
     [Console]::WriteLine("This is an unknown message.")
 }
`

