---
Author: vadims podans
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1364
Published Date: 2010-10-02t04
Archived Date: 2016-06-02t09
---

# test-filelock - 

## Description

simle function to determine is file locked by external program or not.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #####################################################################
 #
 #
 #####################################################################
 
 filter Test-FileLock {
     if ($args[0]) {$filepath = gi $(Resolve-Path $args[0]) -Force} else {$filepath = gi $_.fullname -Force}
     if ($filepath.psiscontainer) {return}
     $locked = $false
     trap {
         Set-Variable -name locked -value $true -scope 1
         continue
     }
     $inputStream = New-Object system.IO.StreamReader $filepath
     if ($inputStream) {$inputStream.Close()}
     @{$filepath = $locked}
 }
`

