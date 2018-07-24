---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4774
Published Date: 2016-01-06t23
Archived Date: 2016-03-18t21
---

# libraryprompt.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##############################################################################
 
 Set-StrictMode -Version Latest
 
 function Prompt
 {
     $id = 1
     $historyItem = Get-History -Count 1
     if($historyItem)
     {
         $id = $historyItem.Id + 1
     }
 
     Write-Host -ForegroundColor DarkGray "`n[$(Get-Location)]"
     Write-Host -NoNewLine "PS:$id > "
     $host.UI.RawUI.WindowTitle = "$(Get-Location)"
 
     "`b"
 }
`

