---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 175
Published Date: 2008-04-15t21
Archived Date: 2012-12-29t04
---

# page-output - 

## Description

this is like a (very simple) “more” script for powershell … the problem with it is that you’re paging by a count of objects, not by how many lines of text they’ll output … so the paging doesn’t really work except for format-table output … unless you specify it manually.  however, this script provides you with “an option” if you want to have paging and still be able to use a script to color the output based on context or syntax.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################################################
 ##
 ###################################################################################################
 ###################################################################################################
 Param( [int]$count=($Host.UI.RawUI.WindowSize.Height-5) )
    BEGIN {
       $x  = 1
       $bg = $Host.UI.RawUI.BackgroundColor;
       $fg = $Host.UI.RawUI.ForegroundColor;
       $page = $true
    }
    PROCESS {
       if($page -and (($x++ % $count) -eq 0)) {
          Write-Host "Press a key to continue (or 'A' for all, or 'Q' to quit)..." -NoNew;
          $Host.UI.RawUI.FlushInputBuffer();
          $ch = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyUp");
          switch -regex ($ch.Character){
             "a|A" { $page = $false }
             "q|Q" { exit }
          }
          $pos = $Host.UI.RawUI.CursorPosition;
          $pos.X = 0;
          $Host.UI.RawUI.CursorPosition = $pos;
       }
       $_
    }
    END {
       $Host.UI.RawUI.BackgroundColor = $bg;
       $Host.UI.RawUI.ForegroundColor = $fg;
    }
 #}
`

