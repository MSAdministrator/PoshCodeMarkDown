---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4600
Published Date: 2013-11-11t14
Archived Date: 2013-11-14t15
---

# posh hex2dec - 

## Description

fixed some issues

## Comments



## Usage



## TODO



## function

`convert-hex2dec`

## Code

`#
 #
 Set-Alias hex2dec Convert-Hex2Dec
 
 function Convert-Hex2Dec {
   <#
     .EXAMPLE
         PS C:\> Convert-Hex2Dec 7717
         7717 = 0x1E25
     .EXAMPLE
         PS C:\> hex2dec fa
         0xFA = 250
     .EXAMPLE
         PS C:\> "0x15" | hex2dec
         0x15 = 21
     .EXAMPLE
         PS C:\> Convert-Hex2Dec x200
         0x200 = 512
   #>
   param(
     [Parameter(Mandatory=$true,
                ValueFromPipeline=$true)]
     [ValidateNotNullOrEmpty()]
     [String]$Number
   )
 
   begin {
     function col($c) {$host.UI.RawUI.ForegroundColor = $c}
     $old = $host.UI.RawUI.ForegroundColor
   } 
   process {
     if ($Number -match "^[0-9]+$") {
       col "Magenta"
       "{0} = 0x{1:X}`n" -f $Number, [Int32]$Number
       col $old
     }
     elseif ($Number -match "^(0x|x)|([0-9]|[a-f])+$") {
       col "Cyan"
       try {
         switch ($Number.Substring(0, 1)) {
           "x" {"0x{0} = {1}`n" -f $Number.Substring(1, $Number.Length - 1).ToUpper(), `
                                                                 [Int32]("0" + $Number)}
           "0" {"0x{0} = {1}`n" -f $Number.Substring(2, $Number.Length - 2).ToUpper(), `
                                                                         [Int32]$Number}
           default {
             "0x{0} = {1}`n" -f $Number.ToUpper(), [Int32]("0x" + $Number)
           }
         }
       }
       catch {}
       col $old
     }
   }
 }
`

