---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 835
Published Date: 2009-01-30t02
Archived Date: 2017-04-30t12
---

# play-note(s) - 

## Description

code to play distinct “musical” notes using console.beep allowing you to specify the note/octave/duration as well as speaking

## Comments



## Usage



## TODO



## function

`play-note`

## Code

`#
 #
 function Play-Note([string]$note,[int] $duration = 5)  
 {  
   if (!($note -match '(\d+)')) { $note+='4' };[void]($note -match '([A-G#]{1,2})(\d+)')  
   [console]::Beep((440 * [math]::Pow([math]::pow(2,(1/12)),
     (([int] $matches[2]) - 4)* 12 + $( switch($matches[1])  
   {  'A'  { 0 }  'A#' { 1 }  'Bb' { 1 }  'B'  { 2 }  'C'  { 3 }  'C#' { 4 }  'Db' { 4 }  
      'D'  { 5 }  'D#' { 6 }  'Eb' { 6 }  'E'  { 7 }  'F'  { 8 }  'F#' { 9 }  'Gb' { 9 }  
      'G'  { 10 } 'G#' { 11 } 'Ab' { 11 }  
   }))),$duration * 100 )    
 }  
 
 function Speak-words ([string]$words,[bool]$pause = $true)
 {   $flag = 1 
     if ($pause) {$flag = 2} 
     $voice = new-Object -com SAPI.spvoice
 }
 
 
 function Play-Notes
 {
 $defaultduration = 5;if($args[0] -is [int]) {$defaultduration = $args[0]}
 for($i = 0;$i -lt $args.length;$i++)
  {    $duration = $defaultduration
     if ($i -lt $args.length-1) { if ($args[$i+1] -is [int]) {$duration = $args[$i+1] } }
     if ($args[$i] -is [string]) { 
         if ($args[$i].startswith("!")) { speak-words $args[$i].replace('!','') } else 
         { play-note $args[$i] $duration }
         }
  }
 }
 
 play-notes A B C 20 E 10 B 10
 play-notes A B C 20 E 10 B 10 
 play-notes F5 2 E5 2 D5 2 C5 2 B5 2 A5 2 G 4 F5 2 E5 2 D5 2 C5 2 B5 2 A5 2 G 4
 play-notes 2 F5 E5 D5 C5 B5 A5 G 4 F5 E5 D5 C5 B5 A5 G 4
 play-notes 4 F5 E5 D5 C5 B5 A5 G F5 E5 D5 C5 B5 A5 G 
 play-notes "!here we go now" Db5 Db Eb Gb Ab "C#" 6 !YEAH
 play-notes A "!$([datetime]::now)" C
`

