---
Author: richard vantreas
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3372
Published Date: 2012-04-18t21
Archived Date: 2012-04-22t08
---

# powershell_ise profile - 

## Description

http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #--------------------------------------------------------------------------------------------------------------
 $psise.CurrentPowerShellTab.Files | % { 
     $_.gettype().getfield("encoding","nonpublic,instance").setvalue($_, [text.encoding]::ascii) 
 } 
 
 register-objectevent $psise.CurrentPowerShellTab.Files collectionchanged -action { 
     $event.sender | % { 
         $_.gettype().getfield("encoding","nonpublic,instance").setvalue($_, [text.encoding]::ascii) 
     } 
 }
 #--------------------------------------------------------------------------------------------------------------
`

