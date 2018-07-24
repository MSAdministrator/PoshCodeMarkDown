---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2151
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# get-detailedsysteminform - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Get detailed information about a system.
 
 .EXAMPLE
 
 Get-DetailedSystemInformation LEE-DESK > output.txt
 Gets detailed information about LEE-DESK and stores the output into output.txt
 
 #>
 
 param(
     $Computer = "."
 )
 
 Set-StrictMode -Version Latest
 
 "#"*80
 "System Information Summary"
 "Generated $(Get-Date)"
 "#"*80
 ""
 ""
 
 "#"*80
 "Computer System Information"
 "#"*80
 Get-WmiObject Win32_ComputerSystem -Computer $computer | Format-List *
 
 "#"*80
 "Operating System Information"
 "#"*80
 Get-WmiObject Win32_OperatingSystem -Computer $computer | Format-List *
 
 "#"*80
 "BIOS Information"
 "#"*80
 Get-WmiObject Win32_Bios -Computer $computer | Format-List *
 
 "#"*80
 "Memory Information"
 "#"*80
 Get-WmiObject Win32_PhysicalMemory -Computer $computer | Format-List *
 
 "#"*80
 "Physical Disk Information"
 "#"*80
 Get-WmiObject Win32_DiskDrive -Computer $computer | Format-List *
 
 "#"*80
 "Logical Disk Information"
 "#"*80
 Get-WmiObject Win32_LogicalDisk -Computer $computer | Format-List *
`

