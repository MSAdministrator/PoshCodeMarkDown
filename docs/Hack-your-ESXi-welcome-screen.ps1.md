---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1947
Published Date: 
Archived Date: 2014-09-03t23
---

#  - 

## Description

hack your esxi welcome screen.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $screen = " You see here a virtual switch.            ------------           ------
                                           #...........|           |....|
                       ---------------   ###------------           |...(|
                                               ---.-----   ###
                                               |.......|####
                                               ---------
 .
  Clyde the Sysadmin    St:7 Dx:9 Co:10 In:18 Wi:18 Ch:6    Chaotic Evil
  Dlvl:3  $:120 HP:39(41) Pw:36(36) AC:6  Exp:5 T:1073
 "
 Set-VMHostAdvancedConfiguration -name Annotations.WelcomeMessage -value $screen
`

