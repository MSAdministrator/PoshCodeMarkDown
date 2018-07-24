---
Author: 129rqw
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2902
Published Date: 2011-08-07t17
Archived Date: 2011-08-22t02
---

# prevent-screensaver - 

## Description

simulate user activity to prevent desktop lock or screensaver for specified period of time

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #########################################################
 #########################################################
 #########################################################
 ########################################################
 ########################################################
 
 param($minutes = 60)
 
 $myshell = New-Object -com "Wscript.Shell"
 
 for ($i = 0; $i -lt $minutes; $i++) {
   Start-Sleep -Seconds 60
   $myshell.sendkeys(".")
 }
`

