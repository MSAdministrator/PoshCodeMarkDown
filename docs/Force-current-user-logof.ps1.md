---
Author: lieuhon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5503
Published Date: 2015-10-10t01
Archived Date: 2015-01-31t20
---

# force current user logof - 

## Description

a simple function to force current user logoff without warning.  the function takes one computer name or a list of computer names.

## Comments



## Usage



## TODO



## function

`exit-currentuser`

## Code

`#
 #
 function exit-CurrentUser
 {
   <#
     .SYNOPSIS
         Force current logon user logoff; be careful, there is no warning will be given to user!!
 
     .EXAMPLE
         Exit-CurrentUser -computer com1
   #>
     param (
         [parameter(
             Mandatory = $true,
             ValueFromPipeline=$true)]
         [string[]]
         $computers
     )
     foreach ($computer in $computers) {
         [Void](gwmi -Class Win32_OperatingSystem -ComputerName $computer).Win32Shutdown(4)
     }
 }
`

