---
Author: florian frank
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4565
Published Date: 2016-10-27t22
Archived Date: 2016-04-20t01
---

# ps - 

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

