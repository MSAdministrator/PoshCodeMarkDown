---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4992
Published Date: 2014-03-17t08
Archived Date: 2014-03-22t00
---

# test-packer - 

## Description

checking file if it has been packed with something like upx.

## Comments



## Usage



## TODO



## function

`test-packer`

## Code

`#
 #
 function Test-Packer {
   <#
     .EXAMPLE
         PS C:\>Test-Packer E:\bin\whois.exe
         File has not been packed.
     .EXAMPLE
         PS C:\>Test-Packer E:\bin\delprof.exe
         File has been packed with: UPX
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName
   )
   
   begin {
     $FileName = cvpa $FileName
   }
   process {
     try {
       $fs = [IO.File]::OpenRead($FileName)
       $buf = New-Object "Byte[]" 1024
       [void]$fs.Read($buf, 0, $buf.Length)
       $e_lfanew = 256 * $buf[0x3D] + $buf[0x3C]
       if ([String]::Join('', [Char[]]$buf[0..1]) -ne 'MZ' -and
         [String]::Join('', [Char[]]$buf[$e_lfanew..($e_lfanew + 3)]) -ne "PE`0`0") {
         throw (New-Object FormatException('Invalid file format.'))
       }
       function private:get([Int32]$offset, [Switch]$band) {
         return $([BitConverter]::ToUInt32($buf, ($e_lfanew + $offset)) -band 0xffff)
       }
       $ptr = $e_lfanew + (get 0x14) + 24
       $chr = [BitConverter]::ToUInt32($buf, ($ptr + 0x24))
       $Sections = for ($i = 0; $i -lt (get 0x6); ++$i) {
         if ($chr -band 0x20 -or $chr -band 0x20000000) {
           [String]::Join('', [Char[]]$buf[$ptr..($ptr + 0x7)])
         }
         $ptr += 40
       }
     }
     catch {
       $_.Exception.Message
     }
     finally {
       if ($fs -ne $null) { $fs.Close() }
     }
   }
   end {
     if ($Sections -eq '.text') {
       "File has not been packed.`n"
     }
     elseif ($Sections -is [Array]) {
       "File has been packed with: $($Sections[0] -replace '\d+|\.', '')`n"
     }
     else {
       $Sections
     }
   }
 }
`

