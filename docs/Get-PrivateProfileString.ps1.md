---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2161
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-privateprofilestring - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Retrieves an element from a standard .INI file
 
 .EXAMPLE
 
 Get-PrivateProfileString c:\windows\system32\tcpmon.ini `
     "<Generic Network Card>" Name
 Generic Network Card
 
 #>
 
 param(
     $Path,
 
     $Category,
 
     $Key
 )
 
 Set-StrictMode -Version Latest
 
 $signature = @'
 [DllImport("kernel32.dll")]
 public static extern uint GetPrivateProfileString(
     string lpAppName,
     string lpKeyName,
     string lpDefault,
     StringBuilder lpReturnedString,
     uint nSize,
     string lpFileName);
 '@
 
 $type = Add-Type -MemberDefinition $signature `
     -Name Win32Utils -Namespace GetPrivateProfileString `
     -Using System.Text -PassThru
 
 $builder = New-Object System.Text.StringBuilder 1024
 $null = $type::GetPrivateProfileString($category,
     $key, "", $builder, $builder.Capacity, $path)
 
 $builder.ToString()
`

