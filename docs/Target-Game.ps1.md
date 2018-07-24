---
Author: dan in philly
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6883
Published Date: 2017-05-04t15
Archived Date: 2017-05-09t05
---

# target game - 

## Description

this was the first game i wrote in basic as a 12-yr old.  here it is again in powershell.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Clear
     $position = $Host.UI.RawUI.CursorPosition
     $height = $Host.UI.RawUI.WindowSize.height
     $bottom = ($height - 2)
 For($y=2; $y -lt $bottom; $y++){
     $Host.UI.RawUI.CursorPosition = $position
     $vertchar = [char]9553
 Write-Host $vertchar}
 
     $point = Get-Random -Minimum 3 $bottom
     $position.y = $point
     $Host.UI.RawUI.CursorPosition = $position
     $pointchar = [char]9571
 Write-Host $pointchar
 
     $position.x = 0
     $position.y = ($height-2)
     $Host.UI.RawUI.CursorPosition = $position
     $btmPos = ($bottom - 1)
 $guess = Read-Host "Guess where the point is (between 2 and $btmPos)"
 
     $horizChar = [char]9552
     $eraseChar = [char]32
     $width = $Host.UI.RawUI.WindowSize.width
 For($x=0; $x -lt ($width - 5); $x++){
     $position = $Host.UI.RawUI.CursorPosition
     $position.x = $x
     $position.y = $guess
     $Host.UI.RawUI.CursorPosition = $position
 Write-Host $horizChar
 sleep -Milliseconds 25
 
     $erase = $x - 1
     If($erase -gt 0){$position.x = $erase}
     $Host.UI.RawUI.CursorPosition = $position
 Write-Host $eraseChar}
 
     $position.x = ($width - 3)
     $position.y = $point
     $Host.UI.RawUI.CursorPosition = $position
 If($guess -eq $point) {Write-Host $point -ForegroundColor Green}
     Else {Write-Host $point -ForegroundColor Red}
 
     $position.x = 0
     $Host.UI.RawUI.CursorPosition = $position
 If($guess -eq $point) {Write-Host "   WINNER!" -ForegroundColor Green}
     Else {Write-Host " $guess is a LOSER" -ForegroundColor Red}
 $repeat = Read-Host " Again (y/n)"
 If($repeat -ne "n" -or $repeat -ne "N")
     {.\target.ps1}
 Else{Exit}
`

