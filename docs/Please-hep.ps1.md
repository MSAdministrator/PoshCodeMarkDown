---
Author: varrum
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5837
Published Date: 2015-04-29t13
Archived Date: 2015-05-01t17
---

# please hep - 

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

