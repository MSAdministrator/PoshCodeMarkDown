---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4868
Published Date: 2014-02-03t13
Archived Date: 2014-02-09t09
---

# get-systeminstalldate - 

## Description

from greg�s repository on github.

## Comments



## Usage



## TODO



## function

`get-systeminstalldate`

## Code

`#
 #
 function Get-SystemInstallDate([String]$Computer = '.') {
   <#
     .NOTES
         Author: greg zakharov
   #>
   
   try {
     [Management.ManagementDateTimeConverter]::ToDateTime(
       ((New-Object Management.ManagementClass(
         [Management.ManagementPath]('\\' + $Computer + '\root\cimv2:Win32_OperatingSystem')
       )).PSBase.GetInstances() | select InstallDate).InstallDate
     )
   }
   catch [Management.Automation.MethodInvocationException] {
     if ($_.Exception) {
       [TimeZone]::CurrentTimeZone.ToLocalTime([DateTime]'1.1.1970').AddSeconds(
         (gp 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion').InstallDate
       )
     }
   }
 }
`

