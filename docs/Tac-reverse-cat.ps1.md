---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 433
Published Date: 2009-06-26t07
Archived Date: 2017-05-22t03
---

# tac (reverse cat) - 

## Description

originally posted by keith hill on microsoft.public.windows.powershell.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 param([string]$path)
 
 $fs = New-Object System.IO.FileStream ((Resolve-Path $path), 'Open', 'Read')
 trap { $fs.Close(); break }
 
 $pos = $fs.Length
 $sb = New-Object System.Text.StringBuilder
 while (--$pos -ge 0) {
     [void]$fs.Seek($pos, 'Begin')
     $ch = [char]$fs.ReadByte()
     if ($ch -eq "`n" -and $sb.Length -gt 0) {
         $sb.ToString().TrimEnd()
         $sb.Length = 0
     }
     [void]$sb.Insert(0, [char]$ch)
 }
 $sb.ToString().TrimEnd()
`

