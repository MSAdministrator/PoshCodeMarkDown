---
Author: nathan estell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6384
Published Date: 2016-06-14t18
Archived Date: 2016-10-18t11
---

# number river - 

## Description

displays a constant stream of numbers and spaces. this can cause patterns to form and can be mesmerizing

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $backGroundColor="Black"
 $foreGroundColor="Green"
 
 for (;;)
 {
 $r=(get-random -max $max).tostring()
 #[int]$firstDigitRN=$randomNumber[0]
 for ($i=0;$i -lt $firstDigitRN;$i++)
 {
 " " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 #>
 
 if ($r.startswith(0))
 {
 "" | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(1))
 {
 " " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(2))
 {
 "  " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(3))
 {
 "   " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(4))
 {
 "    " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(5))
 {
 "     " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(6))
 {
 "      " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(7))
 {
 "       " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(8))
 {
 "        " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 if ($r.startswith(9))
 {
 "         " | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
 
 $r | write-host -backgroundcolor $backGroundColor -foregroundcolor $foreGroundColor -NoNewLine
 }
`

