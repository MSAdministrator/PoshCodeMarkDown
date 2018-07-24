---
Author: naveen sharma
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5967
Published Date: 2016-08-06t13
Archived Date: 2016-05-17t12
---

# simple test with speech - 

## Description

this is a fun script to take test with the help of input boxes.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 ######################################################################
 [Reflection.Assembly]::LoadWithPartialName('System.Speech') | Out-Null
 $S = New-Object System.Speech.Synthesis.SpeechSynthesizer
 $IQq= gc .\IQQ.txt
 $i=1
 $correct=0
 $Incorrect=0
 $per=0
 $S.SpeakAsync("HELLO THERE, HOW ARE YOU Today, I AM GOING TO TAKE YOUR TEST. ")
 $S.RATE= 2
 function CustomInputBox([string] $title, [string] $message, [string] $defaultText) 
 {
    [void][System.Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic')
    $s.SpeakAsync($message) 
    $_userInput = [Microsoft.VisualBasic.Interaction]::InputBox($message, $title)
    return $_userInput
 }
 
 $name = CustomInputBox "User Name" "Please enter your name." ""
 $Age = CustomInputBox "Age" "Please enter Age." ""
 
 foreach($t in $IQq)
 {
    $Question=$t.Split("*")
    $Ans = CustomInputBox "Question No $i" $Question[0] ""
    if( $Ans -eq $Question[1]){$correct++} else {$incorrect++}
    $i++
 }
 
 if($correct -ne 0) {$per=$correct*100/($correct+$incorrect)}
 $S.SpeakAsync(" $name Your Result $per %")
 
 #################################################################
 <#
 Which one of the five is least like the other four?  A Dog B Mouse C Lion D Snake E Elephant*d
 Which number should come next in the series?  1 - 1 - 2 - 3 - 5 - 8 - 13  A 8 B 13 C 21 D 26 E 31*C
 Which one of the five choices makes the best comparison? PEACH is to HCAEP as 46251 is to:    A 25641  B 26451  C 12654  D 51462  E 15264*e
 Mary, who is sixteen years old, is four times as old as her brother. How old will Mary be when she is twice as old as her brother? A 20  B 24  C 25  D 26  E 28*B
 Which one of the numbers does not belong in the following series? 2 - 3 - 6 - 7 - 8 - 14 - 15 - 30   A THREE  B SEVEN  C EIGHT  D FIFTEEN  E THIRTY*C
 #>
`

