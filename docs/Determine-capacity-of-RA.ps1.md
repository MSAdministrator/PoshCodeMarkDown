---
Author: florian frank
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4564
Published Date: 2015-10-27t22
Archived Date: 2015-10-25t13
---

# determine capacity of ra - 

## Description

get the capacity of your installed ram with the win32_physicalmemory wmi class.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 [long]$memory = 0
 
 Get-WmiObject -Class Win32_PhysicalMemory | ForEach-Object -Process { $memory += $_.Capacity }
 
 $memory / 1GB
`

