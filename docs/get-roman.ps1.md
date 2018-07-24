---
Author: luis c
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6818
Published Date: 2017-03-24t19
Archived Date: 2017-04-08t01
---

# get-roman - 

## Description

short powershell module to convert numbers to roman numerals

## Comments



## Usage



## TODO



## function

`get-roman`

## Code

`#
 #
 function get-roman ([int]$myNum)
 {
     if ($myNum -ge 4000 -or $myNum -le 0) 
     {
         "$myNum is not a good one"
     } else {
         $myRomans = [Ordered]@{ M=1000;CM=900;D=500;CD=400;C=100;XC=90;L=50;XL=40;X=10;IX=9;V=5;IV=4;I=1 }
         foreach ($key in $myRomans.Keys)
         {
             while ($myNum -ge  $myRomans.item($key)) 
             {
             }
         }
         $myOut
     }
 }
`

