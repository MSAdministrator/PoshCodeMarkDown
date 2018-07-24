---
Author: dan jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6358
Published Date: 2016-05-25t07
Archived Date: 2016-05-28t02
---

# set-privilege - 

## Description

i thought that it’s impossible. some guy who calls himself gregzakh wrote a demo module called func which allows you to invoke some winapi functions without defining dynamic modules in memory. this module correctly works on powershell v2, 4 and 5 (i have not got powershell v3 to test it). the script below uses func library to set seshutwondprivilege privilege up for current powershell host.

## Comments



## Usage



## TODO



## script

`set-privilege`

## Code

`#
 #
 function Set-Privilege {
   param(
     [Parameter(Position=0)]
     [ValidateRange(2, 35)]
     
     [Parameter(Position=1)]
     [Switch]$Enable = $true
   )
   
   begin {
     $ptr, $null = Get-ProcAddress ntdll RtlAdjustPrivilege
     $RtlAdjustPrivilege = Set-Delegate $ptr `
          '[Action[UInt32, Boolean, Boolean, Text.StringBuilder]]'
     $ret = New-Object Text.StringBuilder
   }
   process {
     $RtlAdjustPrivilege.Invoke($Privilege, $Enable, $false, $ret)
   }
   end {}
 }
`

