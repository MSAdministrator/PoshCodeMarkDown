---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 869
Published Date: 2009-02-12t23
Archived Date: 2015-04-16t00
---

# divide integer - 

## Description

powershell does all dividing by doubles, even integers, so often to simulate a interger division you have to [math]

## Comments



## Usage



## TODO



## function

`divide-integer`

## Code

`#
 #
 function divide-integer ([int]$dividend , [int]$divisor ){ [int]$local:remainder = $Null;return [Math]::DivRem($dividend,$divisor,[ref]$local:remainder);}
 set-alias i/ divide-integer
 
 i/ 10 3
 
 function divide-integerpipe ([int]$divisor )
 { begin { [int]$local:remainder = $Null}
   process { [Math]::DivRem($_ ,$divisor,[ref]$local:remainder); }
 }
 set-alias i\ divide-integerpipe
 10 | i\ 3
 
 1..10 | i\ 3
`

