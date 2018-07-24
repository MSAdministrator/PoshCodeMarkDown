---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4593
Published Date: 2016-11-08t04
Archived Date: 2016-05-29t07
---

# get-hexdump - 

## Description

fixed

## Comments



## Usage



## TODO



## function

`get-hexdump`

## Code

`#
 #
 Set-Alias od Get-HexDump
 
 function Get-HexDump {
   [CmdletBinding(DefaultParameterSetName="FileName", SupportsShouldProcess=$true)]
   param(
     [Parameter(Mandatory=$true,
                Position=0,
                ValueFromPipeline=$true)]
     [ValidateNotNullOrEmpty()]
     [String]$FileName,
     
     [Parameter(Mandatory=$false,
                Position=1)]
     [ValidateRange(0, 65535)]
     [Int32]$BytesToProcess = -1
   )
   
   begin {
     $ofs = ''
     [Int32]$line = 0
   }
   process {
     switch ((Test-Path $FileName)) {
       $true{
         if ($PSCmdlet.ShouldProcess($FileName, "Get hex dump of")) {
           gc -ea 0 -enc Byte $FileName -re 16 -to $BytesToProcess | % {
             '{0:X8} {1, -49} {2}' -f $line++, [String](
               $_ | % {' ' + ('{0:x}' -f $_).PadLeft(2, "0")}
             ), [String](
               $_ | % {
                 if ([Char]::IsLetterOrDigit($_) -or [Char]::IsSymbol($_) `
                   -or [Char]::IsPunctuation($_)) {[Char]$_}
                 else {'.'}
               }
             )
           }
         }
       }
       default{Write-Warning "file not found or does not exist."}
     }
   }
   end {}
 }
`

