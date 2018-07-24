---
Author: ryan grant
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2778
Published Date: 2011-07-08t18
Archived Date: 2016-08-25t18
---

# showui binary clock - 

## Description

uses showui to display a binary clock. hotkeys h, t and d toggle help, time and date text and +/- keys resize. click and drag anywhere to move. double-click to close. this was inspired by boe prox’s post at

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 <#
 .SYNOPSIS
 Displays a Binary Coded Sexagesimal clock using the ShowUI module.
 
 .DESCRIPTION
 This clock displays time using three rows of blocks. The top row represents
 hours, the middle is minutes and the bottom is seconds. Each of the six columns
 represents a binary digit. The values for each digit in the order displayed 
 are: 32 16 8 4 2 1. Adding the values of each of the "On" blocks in a row yields
 the value for the row's corresponding time part.
 
 Press the 'h' key to toggle the Helper values.
 Press the 't' key to toggle the Time text.
 Press the 'd' key to toggle the Date text.
 Pres the '+'/'-' keys to resize the window (pressing Shift is not required).
 
 Click and drag to move the window.
 Double-click the window to close.
 
 This script was inspired by Boe Prox's post at:
 http://learn-powershell.net/2011/07/06/building-a-binary-clock-with-powershell/
 
 .PARAMETER OnColor
 The color of the "On" blocks which represent the 1 digits in a binary number.
 
 This value must be able to convert to a System.Windows.Media.Brush type. (e.g.
 
 
 .PARAMETER OnColor
 The color of the "Off" blocks which represent the 0 digits in a binary number.
 
 This value must be able to convert to a System.Windows.Media.Brush type. (e.g.
 
 
 .PARAMETER GridColor
 The color of the space between the blocks.
 
 This value must be able to convert to a System.Windows.Media.Brush type. (e.g.
 
 
 .PARAMETER TextColor
 The color of the text on the blocks. All text is off by default. Press 'h' for
 Help values, 't' for Time or 'd' for Date.
 
 This value must be able to convert to a System.Windows.Media.Brush type. (e.g.
 
 
 .PARAMETER Topmost
 This switch determines the window's Topmost attribute.
 
 .EXAMPLE
 
 Displays a topmost clock with a white grid, gray off blocks and purple on blocks.
 
 .EXAMPLE
 
 Displays a clock where the on blocks and any text are the only visible elements.
 
 .NOTES
 NAME:     Show-BinaryClock.ps1
 VERSION:  1.0
 DATE:     2011-07-08
 AUTHOR:   Ryan Grant
 #> 
 
 param(
 [switch] $Topmost
 )
 
 Import-Module ShowUI -ErrorAction Stop
 
 $GLOBAL:backColor = @{'0'=$OffColor ;'1'= $OnColor}
     
 $windowParams = @{
     Width = 160
     Height = 90
     WindowStyle = 'None'
     AllowsTransparency = $true
     Background = $GridColor
     Topmost = $Topmost
 }
 
 Window @windowParams -Show `
 -Content {
     UniformGrid -Name ClockGrid -Columns 6 -Margin 2 -Children {
         foreach($part in @('Hour','Minute','Second'))
             {0..5|%{TextBlock -Name "$part$_" -Margin 2 -Foreground $TextColor}}
     }
 } -On_Loaded {
     Register-PowerShellCommand -In '0:0:0.5' -Run -ScriptBlock {
         $time = Get-Date
         
         $vals =  @($time.Hour, $time.Minute, $time.Second)| 
                     %{[convert]::ToString($_,2)}| 
                     %{('0'*(6-$_.ToString().Length))+$_}
 
         foreach($d in 0..5) {
             (Get-Variable "Hour$d").Value.Background = $backColor[[string]$vals[0][$d]]
             (Get-Variable "Minute$d").Value.Background = $backColor[[string]$vals[1][$d]]
             (Get-Variable "Second$d").Value.Background = $backColor[[string]$vals[2][$d]]
         }
 
         'Hour','Minute'|
             %{(Get-Variable ($_+"0")).Value.Text = ("{0:00}" -f $time.$_) * $IsShowTime}
         
         if(!$IsHelpers)
             {(Get-Variable ("Second0")).Value.Text = ("{0:00}" -f $time.Second) * $IsShowTime}
         
         $Hour3.Text = ("{0:00}" -f $time.Month) * $IsShowDate
         $Hour4.Text = ("{0:00}" -f $time.Day) * $IsShowDate
         $Hour5.Text = ($time.Year.ToString().Substring(2,2)) * $IsShowDate
     }
 } -On_KeyDown {
     switch ($_.Key){
         'H' {
                 $IsShowTime = 0
                 $IsHelpers = $IsHelpers -bxor 1
             }
         'T' {
                 $IsHelpers = 0
                 $IsShowTime = $IsShowTime -bxor 1
             }
         'D' {
                 $IsShowDate = $IsShowDate -bxor 1
             }
         {'Add','OemPlus' -contains $_} {
                 $window.Width *= 1.1
                 $window.Height *= 1.1
             }
         {'Subtract','OemMinus' -contains $_} {
                 if($window.Width -gt 50) {
                     $window.Width /= 1.1
                     $window.Height /= 1.1
                 }
             }
     }
     1..5| %{(Get-Variable "Second$_").Value.Text = "{0:00}" -f [math]::Pow(2,(5-$_)) * $IsHelpers}
     
     if(!$IsShowTime) {(Get-Variable "Second0").Value.Text = '32' * $IsHelpers}    
 } `
 -On_MouseDoubleClick {$window.Close()}`
 -On_MouseLeftButtonDown {$window.DragMove()}
`

