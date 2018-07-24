---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 613
Published Date: 
Archived Date: 2011-01-01t11
---

# pause-script &amp; out-more - 

## Description

two functions, one that emulates the pause functionality from cmd.exe, and one that gives similar functionality to more.com.  out-more is especially useful if you’re doing something like “gc somefile.txt | out-more” because it starts outputting text to the screen immediately instead of waiting for the entire file to be read, which is what happens if you do “gc somefile.txt | more”.  out-more can also be used for other objects besides.

## Comments



## Usage



## TODO



## script

`pause-script`

## Code

`#
 #
 #
 
 function Pause-Script {
   param([string]$message = 'Press any key to continue...')
   Write-Host -NoNewLine $message
   $null = $Host.UI.RawUI.ReadKey("NoEcho, IncludeKeyDown")
   Write-Host ""
 }
 
 
 #
 #
 #
 
 function Out-More {
   param ([int]$window_size = ($Host.UI.RawUI.WindowSize.Height - 1))
   $i = 0
 
   foreach ($line in $input) {
     Write-Output $line
     $i += 1
 
     if ($i -eq $window_size) {
       Pause-Script
       $i = 0
     }
   }
 }
`

