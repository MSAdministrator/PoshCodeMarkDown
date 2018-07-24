---
Author: s-1-5-21-2127521
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5812
Published Date: 2015-04-02t18
Archived Date: 2015-04-05t04
---

# convert raw sid to sid - 

## Description

this scripts converts raw sid such as 010500000000000515000000a065cf7e784b9b5fe77c8770091c0100 into a standard sid output such as s-1-5-21-2127521184-1604012920-1887927527-72713

## Comments



## Usage



## TODO



## function

`convert-hextodec`

## Code

`#
 #
 
 function Convert-HEXtoDEC
 {
 param($HEX)
 ForEach ($value in $HEX)
 {
 [string][Convert]::ToInt32($value,16)
 }
 }
 
 function Reassort
 {
 param($chaine)
 $a = $chaine.substring(0,2)
 $b = $chaine.substring(2,2)
 $c = $chaine.substring(4,2)
 $d = $chaine.substring(6,2)
 $d+$c+$b+$a
 }
 
 function ConvertSID
 {
 param($chaine32)
 foreach($chaine in $chaine32) {
     [INT]$SID_Revision = $chaine.substring(0,2)
     [INT]$Identifier_Authority = $chaine.substring(2,2)
     [INT]$Security_NT_Non_unique = Convert-HEXtoDEC(Reassort($chaine.substring(16,8)))
     $chaine1 = $chaine.substring(24,8)
     $chaine2 = $chaine.substring(32,8)
     $chaine3 = $chaine.substring(40,8)
     $chaine4 = $chaine.substring(48,8)
     [string]$MachineID_1=Convert-HextoDEC(Reassort($chaine1))
     [string]$MachineID_2=Convert-HextoDEC(Reassort($chaine2))
     [string]$MachineID_3=Convert-HextoDEC(Reassort($chaine3))
     [string]$UID=Convert-HextoDEC(Reassort($chaine4))
     #"S-1-5-21-" + $MachineID_1 + "-" + $MachineID_2 + "-" + $MachineID_3 + "-" + $UID
     "S-$SID_revision-$Identifier_Authority-$Security_NT_Non_unique-$MachineID_1-$MachineID_2-$MachineID_3-$UID"
     }
 }
`

