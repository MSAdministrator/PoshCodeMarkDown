---
Author: owen08
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3668
Published Date: 2013-09-27t10
Archived Date: 2016-05-11t05
---

# chassis type - 

## Description

ever wonder what kind of chassis your computer thinks it’s running on? have a need to know if you’re rdp’d into a physical or virtual system? this script can probably answer the question for you. this script was adapted to powershell from a vbscript i’ve had for a while. it uses wmi to determine the chassis type and translates the reported numerical code into decipherable text. – thought i would rewrite this code using the switch function rather than elseif. easier to read and less code to write.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $system = Get-WMIObject -class Win32_systemenclosure
 $type = $system.chassistypes
 
 Switch ($Type)
     {
         "1" {"Chassis type is: $Type - Other"}
         "2" {"Chassis type is: $type - Virtual Machine"}
         "3" {"Chassis type is: $type - Desktop"}
         "4" {"Chassis type is: $type - Low Profile Desktop"}
         "5" {"Chassis type is: $type - Pizza Box"}
         "6" {"Chassis type is: $type - Mini Tower"}
         "7" {"Chassis type is: $type - Tower"}
         "8" {"Chassis type is: $type - Portable"}
         "9" {"Chassis type is: $type - Laptop"}
         "10" {"Chassis type is: $type - Notebook"}
         "11" {"Chassis type is: $type - Handheld"}
         "12" {"Chassis type is: $type - Docking Station"}
         "13" {"Chassis type is: $type - All-in-One"}
         "14" {"Chassis type is: $type - Sub-Notebook"}
         "15" {"Chassis type is: $type - Space Saving"}
         "16" {"Chassis type is: $type - Lunch Box"}
         "17" {"Chassis type is: $type - Main System Chassis"}
         "18" {"Chassis type is: $type - Expansion Chassis"}
         "19" {"Chassis type is: $type - Sub-Chassis"}
         "20" {"Chassis type is: $type - Bus Expansion Chassis"}
         "21" {"Chassis type is: $type - Peripheral Chassis"}
         "22" {"Chassis type is: $type - Storage Chassis"}
         "23" {"Chassis type is: $type - Rack Mount Chassis"}
         "24" {"Chassis type is: $type - Sealed-Case PC"}
         Default {"Chassis type is: $type - Unknown"}
      }
`

