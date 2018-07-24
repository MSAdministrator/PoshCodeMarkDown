---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4867
Published Date: 2014-02-03t13
Archived Date: 2014-02-09t09
---

# get-systemuptime - 

## Description

from greg�s repository on github.

## Comments



## Usage



## TODO



## function

`get-systemuptime`

## Code

`#
 #
 function Get-SystemUptime([String]$Computer = '.') {
   <#
     .NOTES
         Author: greg zakharov
   #>
   
   try {
     New-TimeSpan ([Management.ManagementDateTimeConverter]::ToDateTime(
       ((New-Object Management.ManagementClass(
         [Management.ManagementPath]('\\' + $Computer + '\root\cimv2:Win32_OperatingSystem')
       )).PSBase.GetInstances() | select LastBootUpTime).LastBootUpTime
     )) (date) | ft -a
   }
   catch [Management.Automation.MethodInvocationException] {
     Write-Host Access denied. -fo Red
   }
 }
`

