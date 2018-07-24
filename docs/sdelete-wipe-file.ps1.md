---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4800
Published Date: 2014-01-16t12
Archived Date: 2017-04-30t09
---

# sdelete (wipe file) - 

## Description

imitates sysinternals sdelete tool

## Comments



## Usage



## TODO



## function

`remove-fileabnormally`

## Code

`#
 #
 Set-Alias sdelete Remove-FileAbnormally
 
 function Remove-FileAbnormally {
   <#
     .SYNOPSIS
         Wipe a file.
     .EXAMPLE
         PS C:\>sdelete E:\bin\whois.exe
     .NOTES
         Author: greg zakharov
         Mailto: gregzakharov@bk.ru
   #>
   [CmdletBinding()]
   param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName,
     
     [Parameter(Position=1)]
     [ValidateRange(1, 32)]
     [Int32]$PassThru = 3
   )
   
   begin {
     $FileName = cvpa $FileName
     
     $buf = New-Object "Byte[]" 512
     $rng = New-Object Security.Cryptography.RNGCryptoServiceProvider
   }
   process {
     try {
       if ([Security.Principal.WindowsIdentity]::GetCurrent().Name -ne `
                    ([IO.FileInfo]$FileName).GetAccessControl().Owner) {
         throw (New-Object IO.IOException("File is out of current account."))
       }
       
       [IO.File]::SetAttributes($FileName, [IO.FileAttributes]::Normal)
       $sectors = [Math]::Ceiling((([IO.FileInfo]$FileName).Length / 512))
       
       $fs = New-Object IO.FileStream($FileName, [IO.FileMode]::Open, [IO.FileAccess]::Write)
       for ($i = 0; $i -lt $PassThru; $i++) {
         for ($j = 0; $j -lt $sectors; $j++) {
           $rng.GetBytes($buf)
           $fs.Write($buf, 0, $buf.Length)
         }
       }
       $fs.Close()
       
       $stamp = New-Object DateTime($([Int32](date -u %Y) + 23), 1, 1, 0, 0, 0)
       [IO.File]::SetCreationTime($FileName, $stamp)
       [IO.File]::SetLastWriteTime($FileName, $stamp)
       [IO.File]::SetLastAccessTime($FileName, $stamp)
     
       [IO.File]::Delete($FileName)
     }
     catch {
       $exp = $_.Exception.Message;$exp;""
     }
   }
   end {
     if ($exp -eq $null) {
       Write-Host File $FileName has been deleted with $PassThru passes. -fo Magenta
       ""
     }
   }
 }
`

