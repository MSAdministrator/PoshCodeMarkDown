---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4586
Published Date: 2013-11-06t08
Archived Date: 2013-12-05t12
---

# 2 ways (in 1) get uptime - 

## Description

combines two technics to get last boot system uptime

## Comments



## Usage



## TODO



## function

`get-systemuptime`

## Code

`#
 #
 Set-Alias uptime Get-SystemUptime
 
 function Get-SystemUptime {
   <#
     .SYNOPSIS
         Returns uptime of system.
     .DESCRIPTION
         Actually, this demo shows that you don't need an access to Win32_OperatingSystem
         to get system last boot uptime.
   #>
   [CmdletBinding()]
   param()
   
   begin {
     $usr = [Security.Principal.WindowsIdentity]::GetCurrent()
     $res = (New-Object Security.Principal.WindowsPrincipal $usr).IsInRole(
       [Security.Principal.WindowsBuiltInRole]::Administrator
     )
   }
   process {
     switch ($res) {
       $true{
         $wmi = gwmi Win32_OperatingSystem
         New-TimeSpan $wmi.ConvertToDateTime($wmi.LastBootUpTime) (Get-Date)
       }
       $false{[TimeSpan]::FromMilliseconds([Environment]::TickCount)}
     }
   }
   end{}
 }
`

