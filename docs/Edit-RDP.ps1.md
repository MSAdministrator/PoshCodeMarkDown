---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 675
Published Date: 
Archived Date: 2010-08-27t00
---

# edit-rdp - 

## Description

function/script that opens an rdp file for editing using terminal services client.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################################################################################
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
     param(
         [string]$Path = (throw "A path to a RDP file is required.")
     )
 
     if (Test-Path $path) {
         mstsc.exe /edit $path
     } else {
         throw "Path does not exist."
     }
 #}
`

