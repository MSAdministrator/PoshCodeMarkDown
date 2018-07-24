---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6059
Published Date: 2016-10-21t15
Archived Date: 2016-09-09t00
---

# show-cpu - 

## Description

i tweaked greg zakharov’s script to remove the flicker (don’t redraw the background between bars), and then optimized it to only draw what’s necessary…

## Comments



## Usage



## TODO



## function

`show-cpu`

## Code

`#
 #
 function Show-CPU {
   <#
     .EXAMPLE
         Show-CPU
 
         Monitors CPU and shows a green graph until you press a key.
 
     .EXAMPLE
         Show-CPU -Color Red
 
         Sets red color for graph
   #>
   param(
     [Parameter(Position=1)]
     [ConsoleColor]$Color = 'Green',
 
     [ValidateRange(100,30000)]
     [int]$SampleFrequencyMs = 500
   )
   
   begin {
     $Label = "CPU Usage"
     $pc = New-Object Diagnostics.PerformanceCounter("Processor", "% Processor Time", "_Total")
 
     $raw = $host.UI.RawUI
     $old = $raw.WindowPosition
     $con = $raw.WindowSize
     $rec = New-Object Management.Automation.Host.Rectangle $old.X, $old.Y, `
                                         $raw.BufferSize.Width, ($old.Y + 25)
     $buf = $raw.GetBufferContents($rec)
     
     function hbar([int]$left, [int]$top, [int]$right, [ConsoleColor]$color) {
       $pos = $old
       $pos.X += $left
       $pos.Y += $top
       $row = $raw.NewBufferCellArray(@(' ' * ($right-$left)), $color, $color)
       $raw.SetBufferContents($pos, $row)
     }
     
     function msg([int]$left, [int]$top, [String]$text, [ConsoleColor]$foreground, [ConsoleColor]$background) {
       $pos = $old
       $pos.X += $left
       $pos.Y += $top
       $row = $raw.NewBufferCellArray(@($text), $foreground, $background)
       $raw.SetBufferContents($pos, $row)
     }
 
     sleep -m 250
     while ($Host.UI.RawUI.KeyAvailable) {
       $null = $Host.UI.RawUI.ReadKey()
     }
 
   }
   end {
     hbar 0 1 ($con.Width + 1) $Host.UI.RawUI.BackgroundColor
     msg 0 1 $Label 'Gray' 'DarkYellow'
     $Width = ($con.Width + 1) - $Label.Length
     $Next = [Math]::Round($pc.NextValue() * 100 / $Width) + $Label.Length
 
     while (!$Host.UI.RawUI.KeyAvailable) {      
       $Prev = $Next
       $Next = [Math]::Round($pc.NextValue() * 100 / $Width) + $Label.Length
       if($Next -gt $Prev) {
         hbar $Prev 1 $Next $Color
       } elseif($Prev -gt $Next) {
       }
       sleep -m $SampleFrequencyMs
     }
 
     $raw.SetBufferContents($old, $buf)
   }
 
 
 
 }
`

