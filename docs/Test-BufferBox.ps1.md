---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: utf-8
License: cc0
PoshCode ID: 730
Published Date: 2009-12-12t13
Archived Date: 2016-07-15t09
---

# test-bufferbox - 

## Description

a tech demo

## Comments



## Usage



## TODO



## script

`new-box`

## Code

`#
 #
 ####################################################################################################
 ##
 ##
 ##
 ####################################################################################################
 PARAM(   $title = "PowerShell Interactive Buffer Demo" )
 
 $global:fore  = $Host.UI.RawUI.ForegroundColor
 $global:back  = $Host.UI.RawUI.BackgroundColor 
 
 Function New-Box ($Height, $Width, $Title="") {
    $box = &{ "¯¯$Title$('¯'*($Width-($Title.Length+2)))";
             1..($Height - 2) | % {(' ' * $Width)}; 
             ('_' * $Width);
             1..2 | % {(' ' * $Width)}; 
             }
    $boxBuffer = $Host.UI.RawUI.NewBufferCellArray($box,'Green','Black')
    ,$boxBuffer
 }
 
 Function Scroll-Buffer ($origin,$Width,$Height,$Scroll=-1){
    $re = new-object $RECT $origin.x, $origin.y, ($origin.x + $width-2), ($origin.y + $height)
    $origin.Y += $Scroll
    $Host.UI.RawUI.ScrollBufferContents($re, $origin, $re, $blank)
 }
 
 Function Out-Buffer {
 PARAM ($Message, $Foreground = 'White',$Background = 'Black',[switch]$noscroll)
 BEGIN {
    if($Message) {
       $Message | Out-Buffer -Fore $Foreground -Back $Background -NoScript:$NoScroll
    }
 }
 PROCESS { if($_){$Message = $_
 
    if ( -not $NoScroll) {
       Scroll-Buffer $ContentPos ($WindowSize.Width -2) ($WindowSize.Height - 4)
    }
   
    $host.ui.rawui.SetBufferContents(
       $MessagePos,
       $Host.UI.RawUI.NewBufferCellArray( 
          @($message.PadRight($WindowSize.Width)),
          $Foreground,
          $Background)
    )
 }}}
 
 Function Clear-Prompt () {
    $Host.UI.RawUI.SetBufferContents( $PromptPos, $prompt )
    $Host.UI.RawUI.CursorPosition = $CursorPos
 }
 
 ###################################################################################################
 ###################################################################################################
    $RECT  = "system.management.automation.host.rectangle"
    $blank = new-object System.Management.Automation.Host.BufferCell(' ','Gray','Black','complete')
 
    $WindowSize = $Host.UI.RawUI.WindowSize;
    "`n" * $WindowSize.Height
    $ContentPos = $Host.UI.RawUI.WindowPosition;
 
    $Host.UI.RawUI.ForegroundColor = "Yellow"
    $Host.UI.RawUI.BackgroundColor = "Black"
 
    $MessagePos = $ContentPos
    $MessagePos.Y += ($WindowSize.Height - 4)
    
    $PromptPos = $ContentPos
    $PromptPos.X = 0
    $PromptPos.Y += $WindowSize.Height - 2
    $prompt = $Host.UI.RawUI.NewBufferCellArray( @(&{"> "+(" " * ($WindowSize.Width-3))}), "Yellow", "Black")
    $CursorPos = $PromptPos
    $CursorPos.X += 2
 ###################################################################################################
    $noise = Get-Content ($MyInvocation.MyCommand.Path)
    $index = 0; 
    $random = New-Object "Random"
 
 ###################################################################################################
 $Host.UI.RawUI.SetBufferContents($Host.UI.RawUI.WindowPosition, (New-Box ($WindowSize.Height - 1) $WindowSize.Width $title))
 Out-Buffer "  " -Fore DarkGray -Back Cyan
 Out-Buffer "     This is just a demo. " -Fore Black -Back Cyan
 Out-Buffer "     We will stream in the source code of this script ... " -Fore Black -Back Cyan
 Out-Buffer "     And you can type at the bottom while it's running. " -Fore Black -Back Cyan
 Out-Buffer "     Imagine this as an n-way chat application like IRC, except that " -Fore Black -Back Cyan
 Out-Buffer "  FOR THIS PERFORMANCE ONLY: " -Fore Black -Back Cyan
 Out-Buffer "         The part of your chatting friends is played by my source code. " -Fore DarkGray -Back Cyan
 Out-Buffer "  " -Fore DarkGray -Back Cyan
 Out-Buffer "     Press ESC to exit, or enter 'exit' and hit Enter" -Fore Black -Back Cyan
 Out-Buffer "  " -Fore DarkGray -Back Cyan
 Clear-Prompt
 $line=""
 $exit = $false
 while(!$exit){
    while(!$exit -and $Host.UI.RawUI.KeyAvailable) { 
       $char = $Host.UI.RawUI.ReadKey("IncludeKeyUp,IncludeKeyDown,NoEcho"); 
       if($char.KeyDown) {
       switch([int]$char.Character) {
                if($line.Trim() -eq "exit") { $exit = $true; break; }
 ###################################################################################################
 ###################################################################################################
             iex "Out-Buffer `"$line`" -Fore Red";                                     #############
 ###################################################################################################
 ###################################################################################################
             $line = "";
             Clear-Prompt
          }
             $exit = $true; break;
          }
             if($line.Length -gt 0) {
                $line = $line.SubString(0,$($line.Length-1))
             }
             $Host.UI.RawUI.SetBufferContents( $PromptPos, $Host.UI.RawUI.NewBufferCellArray( @(&{ "> $($line)$(' ' * ($WindowSize.Width-3-$line.Length))"}), "Yellow", "Black") )
          }
          }
          default {
             $line += $char.Character
             $Host.UI.RawUI.SetBufferContents( $PromptPos, $Host.UI.RawUI.NewBufferCellArray( @(&{ "> $($line)$(' ' * ($WindowSize.Width-3-$line.Length))"}), "Yellow", "Black") )
          }
       }
       }
    }
    if($random.NextDouble() -gt 0.75) {
       $noise[($index)..($index++)] | Out-Buffer
       if($index -ge $noise.Length){$index = 0}
    }
    sleep -milli 100
 }
 
 $Host.UI.RawUI.ForegroundColor = $fore
 $Host.UI.RawUI.BackgroundColor = $back
`

