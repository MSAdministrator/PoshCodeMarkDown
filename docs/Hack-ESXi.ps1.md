---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5389
Published Date: 2014-08-30t19
Archived Date: 2014-09-02t00
---

# hack esxi - 

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

