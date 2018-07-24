---
Author: geoff guynn
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6328
Published Date: 2016-04-28t07
Archived Date: 2016-09-08t22
---

# ascii memes - 

## Description

another silly toy i threw together in a few minutes. use get-asciifromstring to generate your own if you want.

## Comments



## Usage



## TODO



## function

`show-asciiface`

## Code

`#
 #
     function Show-AsciiFace{
         param(
             $Name = "Shrug",
             [switch]$Clipboard
         )
         
         $OutputEncoding = [System.Text.Encoding]::unicode
         
         switch ($Name){
             {$_ -like "Shrug*"}{$returnString = $([char[]](0xAF,0x5C,0x5F,0x28,0x30C4,0x29,0x5F,0x2F,0xAF)) -join ""}
             {$_ -like "Flip*"}{$returnString = $([char[]](0x28,0x256F,0xB0,0x25A1,0xB0,0xFF09,0x256F,0xFE35,0x253B,0x2501,0x2501,0x253B)) -join ""}
             {$_ -like "DoubleFlip*"}{$returnString = $([char[]](0x253B,0x2501,0x2501,0x253B,0xFE35,0x20,0x5C,0x28,0xB0,0x25A1,0xB0,0x29,0x2F,0x20,0xFE35,0x20,0x253B,0x2501,0x2501,0x253B)) -join ""}
             {$_ -like "Fix*"}{$returnString = $([char[]](0x252C,0x2500,0x2500,0x252C,0x20,0xFF89,0x28,0xB0,0x2014,0xB0,0xFF89,0x29)) -join ""}
             {$_ -like "Sunglasses*"}{$returnString = $([char[]](0x28,0x20,0x2022,0x5F,0x2022,0x29,0x20,0x28,0x20,0x2022,0x5F,0x2022,0x29,0x3E,0x2310,0x25A0,0x2D,0x25A0,0x20,0x28,0x2310,0x25A0,0x5F,0x25A0,0x29)) -join ""}
             {$_ -like "Disapprove*"}{$returnString = $([char[]](0xCA0,0x5F,0xCA0)) -join ""}
             {$_ -like "Angry*"}{$returnString = $([char[]](0x28,0x22DF,0xFE4F,0x22DE,0x29)) -join ""}
             {$_ -like "You*"}{$returnString = $([char[]](0x261E,0x28,0xFF9F,0x2200,0xFF9F,0x29,0x261E)) -join ""}
             {$_ -like "Gimmie*"}{$returnString = $([char[]](0xF3C,0x20,0x3064,0x20,0x25D5,0x5F,0x25D5,0x20,0xF3D,0x3064)) -join ""}
             {$_ -like "Yay*"}{$returnString = $([char[]](0x5C,0x28,0x2C6,0x2DA,0x2C6,0x29,0x2F)) -join ""}
             {$_ -like "ManTears*"}{$returnString = $([char[]](0xCA5,0x5F,0xCA5)) -join ""}
             {$_ -like "KeepAnEyeOut*"}{$returnString = $([char[]](0x28,0xCA0,0x5F,0x78,0x29,0x20,0xF3C,0x2609)) -join ""}
             {$_ -like "Fight*"}{$returnString = $([char[]](0x28,0xE07,0x2022,0x5F,0x2022,0x29,0xE07)) -join ""}
             {$_ -like "Lenny*"}{$returnString = $([char[]](0x28,0x20,0x361,0xB0,0x20,0x35C,0x296,0x20,0x361,0xB0,0x29)) -join ""}
             default{
                 write-Host "I don't recognize $Name"
                 $returnString = $([char[]](0xAF,0x5C,0x5F,0x28,0x30C4,0x29,0x5F,0x2F,0xAF)) -join ""
             }
         }
         
         if ($Clipboard.IsPresent -eq $True){$returnstring | clip}
         else{return $returnString}
     }
 
     function Get-AsciiFromString{
         param(
             $InputString
         )
         
         return "0x$(($([int[]][char[]]("$InputString")) | % {"{0:X0}" -f $_}) -join ",0x")"
     }
`

