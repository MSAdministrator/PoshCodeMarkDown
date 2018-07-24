---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4735
Published Date: 2016-12-24t08
Archived Date: 2016-07-30t12
---

# get-processorgraph - 

## Description

previous version of this script just builds a graph of processor utilization into powershell host, but how about something more dynamic and customizable, mmm? so i rewrote my get-processorgraph function.

## Comments



## Usage



## TODO



## function

`get-processorgraph`

## Code

`#
 #
 Set-Alias gpg Get-ProcessorGraph
 
 function Get-ProcessorGraph {
   <#
     .EXAMPLE
         PS C:\>Get-ProcessorGraph
         Monitors CPU fifteen seconds.
     .EXAMPLE
         PS C:\>gpg -g Red -w 25
         Sets red color for graph and monitors CPU 25 seconds.
   #>
   param(
     [Parameter(Position=0)]
     [Alias("w")]
     [ValidateRange(10, 1000000)]
     [Int32]$WatchTime = 15,
     
     [Parameter(Position=1)]
     [ConsoleColor]$GraphColor = 'Green'
   )
   
   begin {
     $pc = New-Object Diagnostics.PerformanceCounter("Processor", "% Processor Time", "_Total")
 
     $raw = $host.UI.RawUI
     $old = $raw.WindowPosition
     $con = $raw.WindowSize
     $rec = New-Object Management.Automation.Host.Rectangle $old.X, $old.Y, `
                                         $raw.BufferSize.Width, ($old.Y + 25)
     $buf = $raw.GetBufferContents($rec)
     
     function strip([Int32]$x, [Int32]$y, [Int32]$z, [ConsoleColor]$bc) {
       $pos = $old;$pos.X += $x;$pos.Y += $y
       $row = $raw.NewBufferCellArray(@(' ' * $z), $bc, $bc)
       $raw.SetBufferContents($pos, $row)
     }
     
     function msg([Int32]$x, [Int32]$y, [String]$text, [ConsoleColor]$fc, [ConsoleColor]$bc) {
       $pos = $old;$pos.X += $x;$pos.Y += $y
       $row = $raw.NewBufferCellArray(@($text), $fc, $bc)
       $raw.SetBufferContents($pos, $row)
     }
   }
   process {
     $i = $WatchTime
     while ($i -gt 0) {
       msg 0 1 "CPU Usage" 'Gray' 'DarkYellow'
       strip 9 1 ([Math]::Round(($pc.NextValue() * 100 / $con.Width + 1))) $GraphColor
       sleep -s 1
       $i--
       $raw.SetBufferContents($old, $buf)
     }
   }
   end {
   }
 }
`

