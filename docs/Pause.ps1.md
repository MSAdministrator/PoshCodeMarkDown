---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3339
Published Date: 2012-04-10t11
Archived Date: 2012-04-14t01
---

# pause - 

## Description

function was originally posted by the powershell team on 2007/02/25 @ http

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Pause ($Message = "Press any key to continue...")
 {
 	Write-Host -NoNewline $Message
 	$null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
 	Write-Host ""
 }
`

