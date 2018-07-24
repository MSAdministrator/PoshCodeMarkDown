---
Author: lazywinadmincom
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2760
Published Date: 2011-06-29t19
Archived Date: 2015-10-18t23
---

# services auto notrunning - 

## Description

check if all the services with startmode automatic are actually running

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-WmiObject Win32_Service -ComputerName . |`
 	where 	{($_.startmode -like "*auto*") -and `
 		($_.state -notlike "*running*")}|`
 	select DisplayName,Name,StartMode,State|ft -AutoSize
`

