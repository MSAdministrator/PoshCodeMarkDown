---
Author: nigel stuke
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6374
Published Date: 2016-06-08t17
Archived Date: 2016-06-16t09
---

# logoff - 

## Description

this is a little script i wrote to logoff all users on the box except for myself. clearly it can be cleaned up a little, making it more flexible, but thought i would share anyways.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function RemoveSpace([string]$text) {  
     $private:array = $text.Split(" ", `
     [StringSplitOptions]::RemoveEmptyEntries)
     [string]::Join(" ", $array) }
 
 $quser = quser
 foreach ($sessionString in $quser) {
     $sessionString = RemoveSpace($sessionString)
     $session = $sessionString.split()
     
     if ($session[0].Equals(">nistuke")) {
     continue }
     if ($session[0].Equals("USERNAME")) {
     continue }
     $result = logoff $session[1] }
`

