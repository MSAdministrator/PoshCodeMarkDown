---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2962
Published Date: 2011-09-21t09
Archived Date: 2011-11-05t18
---

# get-freeram - 

## Description

get the free ram from a system

## Comments



## Usage



## TODO



## function

`get-freeram`

## Code

`#
 #
 function Get-FreeRam {
 #.Synopsis
 #.Parameter ComputerName
 #.Example
 #
 [CmdletBinding()]
 param(
   [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
   [string[]]$ComputerName='localhost'
 )
 process {
   Get-WmiObject -ComputerName $ComputerName Win32_OperatingSystem |
   Select-Object -Property @{name="Computer";expression={$_.__SERVER}}, FreePhysicalMemory
 }
 }
`

