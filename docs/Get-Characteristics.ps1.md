---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2149
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t07
---

# get-characteristics.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## class

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
 
 Get the file characteristics of a file in the PE Executable File Format.
 
 .EXAMPLE
 
 Get-Characteristics $env:WINDIR\notepad.exe
 IMAGE_FILE_LOCAL_SYMS_STRIPPED
 IMAGE_FILE_RELOCS_STRIPPED
 IMAGE_FILE_EXECUTABLE_IMAGE
 IMAGE_FILE_32BIT_MACHINE
 IMAGE_FILE_LINE_NUMS_STRIPPED
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [string] $Path
 )
 
 Set-StrictMode -Version Latest
 
 $characteristics = @{}
 $characteristics["IMAGE_FILE_RELOCS_STRIPPED"] = 0x0001
 $characteristics["IMAGE_FILE_EXECUTABLE_IMAGE"] = 0x0002
 $characteristics["IMAGE_FILE_LINE_NUMS_STRIPPED"] = 0x0004
 $characteristics["IMAGE_FILE_LOCAL_SYMS_STRIPPED"] = 0x0008
 $characteristics["IMAGE_FILE_AGGRESSIVE_WS_TRIM"] = 0x0010
 $characteristics["IMAGE_FILE_LARGE_ADDRESS_AWARE"] = 0x0020
 $characteristics["RESERVED"] = 0x0040
 $characteristics["IMAGE_FILE_BYTES_REVERSED_LO"] = 0x0080
 $characteristics["IMAGE_FILE_32BIT_MACHINE"] = 0x0100
 $characteristics["IMAGE_FILE_DEBUG_STRIPPED"] = 0x0200
 $characteristics["IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP"] = 0x0400
 $characteristics["IMAGE_FILE_NET_RUN_FROM_SWAP"] = 0x0800
 $characteristics["IMAGE_FILE_SYSTEM"] = 0x1000
 $characteristics["IMAGE_FILE_DLL"] = 0x2000
 $characteristics["IMAGE_FILE_UP_SYSTEM_ONLY"] = 0x4000
 $characteristics["IMAGE_FILE_BYTES_REVERSED_HI"] = 0x8000
 
 $fileBytes = Get-Content $path -ReadCount 0 -Encoding byte
 
 $signatureOffset = $fileBytes[0x3c]
 
 $signature = [char[]] $fileBytes[$signatureOffset..($signatureOffset + 3)]
 if([String]::Join('', $signature) -ne "PE`0`0")
 {
     throw "This file does not conform to the PE specification."
 }
 
 $coffHeader = $signatureOffset + 4
 
 $characteristicsData = [BitConverter]::ToInt32($fileBytes, $coffHeader + 18)
 
 foreach($key in $characteristics.Keys)
 {
     $flag = $characteristics[$key]
     if(($characteristicsData -band $flag) -eq $flag)
     {
         $key
     }
 }
`

