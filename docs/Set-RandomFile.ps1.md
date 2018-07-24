---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4611
Published Date: 2013-11-16t15
Archived Date: 2013-11-22t23
---

# set-randomfile - 

## Description

creates random file with extension which has assosiation (just for fun).

## Comments



## Usage



## TODO



## function

`set-randomfile`

## Code

`#
 #
 function Set-RandomFile {
   param(
     [Parameter(Mandatory=$true,
                Position=0)]
     [String]$Path,
     
     [Parameter(Position=1)]
     [ValidateRange(0, 31)]
     [Int32]$Length = 7
   )
   
   begin {
     $ext = cmd /c assoc | % {if ($_ -match '=\w+' -and $_ -notmatch '.\d+=') {$_.Split('=')[0]}}
     $itm = -join ([GUID]::NewGuid().Guid -replace '-', '')[0..$Length]
   }
   process {
     $rnd = Get-Random -max ($ext.Length - 1)
   }
   end{
     if (Test-Path $Path) {
       [void](ni -p $Path -n $($itm + $ext[$rnd]) -t File -fo)
     }
   }
 }
`

