---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6711
Published Date: 2017-01-24t11
Archived Date: 2017-01-30t05
---

# reading rar - 

## Description

the main goal of this function to not use additional libraries.

## Comments



## Usage



## TODO



## function

`get-rarcontent`

## Code

`#
 #
 function Get-RarContent {
   <#
     .SYNOPSIS
         Gets content of a compressed (RAR) file.
     .DESCRIPTION
         The main goal of this function to not use additional binary
         files such as WinRAR.exe and etc.
     .NOTES
         Author: greg zakharov
     .LINKS
         https://github.com/gregzakh/alt-ps/blob/master/tools/Get-RarContent.ps1
   #>
   param(
     [Parameter(Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [ValidateScript({Test-Path $_})]
     [String]$Path
   )
 
   begin {
     [Byte[]]$rar = 0x52, 0x61, 0x72, 0x21, 0x1A, 0x07, 0x00
   }
   process {
     try {
       $fs = [IO.File]::OpenRead((Convert-Path $Path))
       $br = New-Object IO.BinaryReader($fs)
 
       if (($cmp = Compare-Object ($br.ReadBytes(7)) $rar -PassThru)) {
         throw New-Object IO.IOException(
           'Specified file is nat a RAR archive, try another.'
         )
       }
       if (($mhd = $br.ReadBytes(7))[2] -ne 0x73) {
         throw New-Object IO.IOException(
           'Required structure MAIN_HEAD has not been found.'
         )
       }
       [void]$fs.Seek((
         [BitConverter]::ToUInt16($mhd[5..6], 0) - 7
       ), [IO.SeekOrigin]::Current)
       $content = while ($true) {
         if (($bsz = [BitConverter]::ToUInt16($seg[5..6], 0)) -le 7) {
         }
 
         if ($seg[2] -eq 0x74) {
           [void]$fs.Seek((
           ), [IO.SeekOrigin]::Current)
 
           if ((
             $atr = [BitConverter]::ToUInt32($seg[28..31], 0)
           ) -band 0x10 -or $atr -band 0x4000) {
           }
 
           New-Object PSObject -Property @{
             Name = -join [Char[]]$seg[32..(
               32 + [BitConverter]::ToUInt16($seg[26..27], 0
             ) - 1)]
             Cmpd = $cmp
             Size = [BitConverter]::ToUInt32($seg[11..14], 0)
             Attr = -join (([IO.FileAttributes]$atr).ToString().Split(
               ', ', [StringSplitOptions]::RemoveEmptyEntries
             ) | ForEach-Object { $_[0] })
           }
         }
         else {
           if ([BitConverter]::ToUInt16($seg[3..4], 0) -band 0x8000) {
             [void]$fs.Seek([BitConverter]::ToUInt32(
               $seg[7..10], 0), [IO.SeekOrigin]::Current
             )
           }
         }
     }
     catch { Write-Warning $_ }
     finally {
       if ($br) { $br.Close() }
       if ($fs) { $fs.Dispose() }
     }
   }
   end {
     if ($content) {
       $content | Select-Object Attr, Size, Cmpd, Name |
       Format-Table -AutoSize
     }
   }
 }
`

