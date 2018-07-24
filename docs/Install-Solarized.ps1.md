---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3080
Published Date: 2012-12-04t10
Archived Date: 2016-06-10t16
---

# install-solarized - 

## Description

convert a console shortcut (e.g.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 [CmdletBinding()]
 param($Path = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Accessories\Windows PowerShell\Windows PowerShell.lnk" )
 
 
 
 
 $lnk = Get-Link $Path
 
 $lnk.ConsoleColors[0]  = $Base03
 $lnk.ConsoleColors[1]  = $Base02
 $lnk.ConsoleColors[2]  = $Base01
 $lnk.ConsoleColors[3]  = $Base00
 $lnk.ConsoleColors[4]  = $Base0
 $lnk.ConsoleColors[5]  = $Violet
 $lnk.ConsoleColors[6]  = $Orange
 $lnk.ConsoleColors[7]  = $Base2
 $lnk.ConsoleColors[8]  = $Base1
 $lnk.ConsoleColors[9]  = $Blue
 $lnk.ConsoleColors[10] = $Green
 $lnk.ConsoleColors[11] = $Cyan
 $lnk.ConsoleColors[12] = $Red
 $lnk.ConsoleColors[13] = $Magenta
 $lnk.ConsoleColors[14] = $Yellow
 $lnk.ConsoleColors[15] = $Base3
 
 
 $lnk.Save()
`

