---
Author: david johnson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4531
Published Date: 2013-10-18t18
Archived Date: 2013-10-21t07
---

# shuffle-string - 

## Description

returns a shuffled / randomised / randomized version of an input string

## Comments



## Usage



## TODO



## function

`shuffle-string`

## Code

`#
 #
 Function Shuffle-String ([string]$String, [switch]$IgnoreSpaces, [switch]$IgnoreCRLF, [switch]$IgnoreWhitespace) {
 ##
 
 
     If ($string.length -eq 0) 
     {
         Return
     }
     
     
     If ($IgnoreWhiteSpace)
     {
         $String = $String.Replace([string][char]9,"")
         $IgnoreSpaces = $True
     }
 
     If ($IgnoreSpaces)
     {
         $String = $String.Replace(" ","")
     }
 
 
     If ($IgnoreCRLF)
     {
         $String = $String.Replace([string][char]10,"").Replace([string][char]13,"")
     }
 
     $Random = New-Object Random
     
     Return [string]::join("",($String.ToCharArray()|sort {$Random.Next()}))
 
 }
`

