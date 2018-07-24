---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5779
Published Date: 
Archived Date: 2016-03-22t05
---

#  - 

## Description

can someone explain to me what the following code is used to do? and how many it will produce? and how would the output be used?

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $chars = "b","c","d","f","g","h","j","k","l","m","n","p","q","r","s","t","v","w","x","y","z" 
 
 foreach($char1 in $chars){ 
 foreach($char2 in $chars){ 
 foreach($char3 in $chars){ 
 foreach($char4 in $chars){ 
 $pw = $char1+$char2+$char3+$char4 
 write-host $pw 
 } 
 } 
 }	
 }
`

