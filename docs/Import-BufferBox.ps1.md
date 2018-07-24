---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 110
Published Date: 2008-01-13t16
Archived Date: 2015-12-13t00
---

# import-bufferbox - 

## Description

a bunch of script functions for creating a simple in-console split-view with output above and an input line below. it allows scripts to sort-of simulate accepting input while they output text. it’s still pretty fragile and occasionally does weird things while you’re typing, because it’s not really multi-threaded and the $host doesn’t have a “lineavailable” method … and if you pause a script, you can’t see output from typing, so the whole things is well and truly a hack. but it works!

## Comments



## Usage



## TODO



## script

`test-bufferbox`

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 ##
 ###################################################################################################
 
 $global:_RECTANGLE_  = "system.management.automation.host.rectangle"
 $global:_BLANKCELL_ = new-object System.Management.Automation.Host.BufferCell(' ','Black','Black','complete')
 
 Function global:New-BufferBox ($Height, $Width, $Title="") {
    $box = &{"Â¯Â¯$Title$('Â¯'*($Width-($Title.Length+2)))";
             1..($Height - 2) | % {(' ' * $Width)}; 
             ('_' * $Width);
             1..2 | % {(' ' * $Width)}; 
             }
    $boxBuffer = $Host.UI.RawUI.NewBufferCellArray($box,'Green','Black')
    ,$boxBuffer
 }
 
 Function global:Move-BufferBox ($Origin,$Width,$Height,$Scroll=-1){
    $re = new-object $_RECTANGLE_ $origin.x, $origin.y, ($origin.x + $width-2), ($origin.y + $height)
    $origin.Y += $Scroll
    $Host.UI.RawUI.ScrollBufferContents($re, $origin, $re, $_BLANKCELL_)
 }
 
 Function global:Write-Message ($Message,$Foreground = 'White',$Background = 'Black',[switch]$noscroll) {
    if ( -not $NoScroll) {
       Move-BufferBox $ContentPos ($WindowSize.Width -2) ($WindowSize.Height -5)
    }
   
    $host.ui.rawui.SetBufferContents(
       $MessagePos,
       $Host.UI.RawUI.NewBufferCellArray( 
          @($message.PadRight($WindowSize.Width)),
          $Foreground,
          $Background)
    )
 }
 
 Function global:Clear-PromptBox {
    $Host.UI.RawUI.SetBufferContents( $PromptPos, $prompt )
 }
 
 Function global:Initialize-BufferBox($Title) {
    ###################################################################################################
    ###################################################################################################
    $script:WindowSize = $Host.UI.RawUI.WindowSize;
    "`n" * $WindowSize.Height
    $script:ContentPos = $Host.UI.RawUI.WindowPosition;
 
    $Host.UI.RawUI.SetBufferContents($ContentPos, (New-BufferBox ($WindowSize.Height - 2) $WindowSize.Width $title))
 
    $script:MessagePos = $ContentPos
    $MessagePos.Y += ($WindowSize.Height - 5)
    
    $script:PromptPos = $ContentPos
    $PromptPos.X = 0
    $PromptPos.Y += $WindowSize.Height - 3
    $script:prompt = $Host.UI.RawUI.NewBufferCellArray( @(&{" " * $WindowSize.Width;" " * $WindowSize.Width}), "Yellow", "Black")
 }
 
 
 Function Test-BufferBox {
    $fore = $Host.UI.RawUI.ForegroundColor
    $back = $Host.UI.RawUI.BackgroundColor 
 
    $Host.UI.RawUI.ForegroundColor = "Yellow"
    $Host.UI.RawUI.BackgroundColor = "Black"
    
    Initialize-BufferBox "Testing BufferBox"
 
    Write-Message 'Welcome to the BufferBox script by Joel "Jaykul" Bennett'
    Write-Message "With great inspiration from /\/\o\/\/ http://ThePowerShellGuy.com"
    Write-Message "You're about to see text fly by up here in the top of the window"
    Write-Message "But while it's flying, you can still type down in the bottom!"
    Write-Message "Try it out. Press any key to start."
    $Host.UI.RawUI.ReadKey( "IncludeKeyDown" ) | out-null
    [string]$line=""
    Get-Content $MyInvocation.ScriptName |  foreach {
       Write-Message $_
       while($Host.UI.RawUI.KeyAvailable) {
          $k = $Host.UI.RawUI.ReadKey( "IncludeKeyUp" )
          if($k.VirtualKeyCode -eq 13 )
          {
             Write-Message $line -fore red -back yellow
             $line=""
             Clear-PromptBox
          }
          elseif($k.VirtualKeyCode -eq 8 )
          {
             $line = $line.SubString(0,$($line.Length-2))
          }
          elseif($k.Character -ne 0) 
          {
             $line += $k.Character
          }
       }
       1..10 | % {[System.Threading.Thread]::Sleep(20)}
    }
    Write-Message "Thanks for playing..."
    [System.Threading.Thread]::Sleep(1000)
   
    $Host.UI.RawUI.ForegroundColor = $fore
    $Host.UI.RawUI.BackgroundColor = $back
    $Host.UI.RawUI.FlushInputBuffer()
 }
 
 if($args.Count -gt 0) { Test-BufferBox }
`

