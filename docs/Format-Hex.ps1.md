---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2143
Published Date: 2010-09-09t21
Archived Date: 2016-10-30t01
---

# format-hex.ps1 - 

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
 
 Outputs a file or pipelined input as a hexadecimal display. To determine the
 offset of a character in the input, add the number at the far-left of the row
 with the the number at the top of the column for that character.
 
 .EXAMPLE
 
 "Hello World" | Format-Hex
 
             0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
 
 00000000   48 00 65 00 6C 00 6C 00 6F 00 20 00 57 00 6F 00  H.e.l.l.o. .W.o.
 00000010   72 00 6C 00 64 00                                r.l.d.
 
 .EXAMPLE
 
 Format-Hex c:\temp\example.bmp
 
 #>
 
 [CmdletBinding(DefaultParameterSetName = "ByPath")]
 param(
     [Parameter(ParameterSetName = "ByPath", Position = 0)]
     [string] $Path,
 
     [Parameter(
         ParameterSetName = "ByInput", Position = 0,
         ValueFromPipeline = $true)]
     [Object] $InputObject
 )
 
 begin
 {
     Set-StrictMode -Version Latest
 
     [byte[]] $inputBytes = $null
     if($Path) { $inputBytes = [IO.File]::ReadAllBytes((Resolve-Path $Path)) }
 
     $counter = 0
     $header = "            0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F"
     $nextLine = "{0}   " -f  [Convert]::ToString(
         $counter, 16).ToUpper().PadLeft(8, '0')
     $asciiEnd = ""
 
     "`r`n$header`r`n"
 }
 
 process
 {
     if(Test-Path variable:\InputObject)
     {
         if($InputObject -is [Byte])
         {
             $inputBytes = $InputObject
         }
         else
         {
             $inputString = [string] $InputObject
             $inputBytes = [Text.Encoding]::Unicode.GetBytes($inputString)
         }
     }
 
     foreach($byte in $inputBytes)
     {
         $nextLine += "{0:X2} " -f $byte
 
         if(($byte -ge 0x20) -and ($byte -le 0xFE))
         {
             $asciiEnd += [char] $byte
         }
         else
         {
             $asciiEnd += "."
         }
 
         $counter++;
 
         if(($counter % 16) -eq 0)
         {
 
             "$nextLine $asciiEnd"
             $nextLine = "{0}   " -f [Convert]::ToString(
                 $counter, 16).ToUpper().PadLeft(8, '0')
             $asciiEnd = "";
         }
     }
 }
 
 end
 {
     if(($counter % 16) -ne 0)
     {
         while(($counter % 16) -ne 0)
         {
             $nextLine += "   "
             $asciiEnd += " "
             $counter++;
         }
         "$nextLine $asciiEnd"
     }
 
     ""
 }
`

