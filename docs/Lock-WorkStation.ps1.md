---
Author: mike pfeiffer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6037
Published Date: 2016-10-05t10
Archived Date: 2016-05-17t13
---

# lock-workstation - 

## Description

locks the workstation’s display. locking a workstation protects it from unauthorized use.

## Comments



## Usage



## TODO



## function

`lock-workstation`

## Code

`#
 #
 Function Lock-WorkStation {
 $signature = @"
 [DllImport("user32.dll", SetLastError = true)]
 public static extern bool LockWorkStation();
 "@
 
 $LockWorkStation = Add-Type -memberDefinition $signature -name "Win32LockWorkStation" -namespace Win32Functions -passthru
 $LockWorkStation::LockWorkStation() | Out-Null
 }
`

