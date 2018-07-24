---
Author: jason archer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 674
Published Date: 2009-11-17t00
Archived Date: 2015-11-03t16
---

# new-rdp - 

## Description

function/script that creates a new rdp file for terminal services.  nothing fancy though.

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
 #
 #
 #
 #
 #
 #
 
     param(
         [string]$Path = $(throw "A path to the new RDP file is required."),
         [string]$Server = "",
         [switch]$PassThru,
         [switch]$Force
     )
 
     if (!(Test-Path $path) -or $force) {
         Set-Content -Path $path -Value "full address:s:$server" -Force
         if ($passthru) {
             Get-ChildItem -Path $path
         }
     } else {
         throw "Path already exists, use the -Force switch."
     }
 #}
`

