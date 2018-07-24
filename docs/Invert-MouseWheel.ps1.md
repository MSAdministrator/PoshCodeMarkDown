---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6734
Published Date: 2017-02-15t06
Archived Date: 2017-02-18t06
---

# invert-mousewheel - 

## Description

inverts the mouse wheel scrolling in windows (to match the way it works in os x)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Enum\HID\*\*\Device` Parameters FlipFlopWheel -EA 0 | 
 ForEach-Object { Set-ItemProperty $_.PSPath FlipFlopWheel 1 }
`

