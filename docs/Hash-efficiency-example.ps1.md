---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6283
Published Date: 2016-04-06t23
Archived Date: 2016-11-18t17
---

# hash efficiency example - 

## Description

tldr

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $rng = 10000
 (Measure-Command {
     $hash = @{}
     foreach ($a in 1..$rng){
         $hash[$a] = $a
     }
 }).totalmilliseconds
 
 (Measure-Command {
     $hash = @{}
     foreach ($a in 1..$rng){
         $hash.$a = $a
     }
 }).TotalMilliseconds
 
 (Measure-Command {
     $hash = @{}
     foreach ($a in 1..$rng){
         $hash.add($a, $a)
     }
 }).TotalMilliseconds
`

