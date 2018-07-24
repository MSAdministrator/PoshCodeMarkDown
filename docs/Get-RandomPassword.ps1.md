---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5806
Published Date: 2015-04-01t18
Archived Date: 2015-04-03t14
---

# get-randompassword - 

## Description

powershell function i made to generate a random password that enforces one of each of upper, lower, and number characters, with optional special characters.

## Comments



## Usage



## TODO



## function

`get-randompassword`

## Code

`#
 #
 Function Get-RandomPassword {
   Param(
     [parameter()]
     [ValidateRange(8,100)]
     [int]$Length=12,
 
     [parameter()]
     [switch]$SpecialChars
 
   )
     $aUpper      = [char[]]'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
     $aLower      = [char[]]'abcdefghijklmnopqrstuvwxyz'
     $aNumber     = [char[]]'1234567890'
     $thePass     = ""
     if ($SpecialChars){
         $aClasses += 'S'
     }
     $notOneOfEach = $true
     while ($notOneOfEach){
         $pPattern = ''
         for ($i=0; $i-lt $Length; $i++){
             $pPattern += get-random -InputObject $aClasses
         }
         if (($pPattern -match 'U' -and $pPattern -match 'L' -and $pPattern -match 'N' -and (-not $specialChars)) -or ($pPattern -match 'U' -and $pPattern -match 'L' -and $pPattern -match 'N' -and $pPattern -match 'S')){
             $notOneOfEach = $false
         }
     }
     foreach ($chr in [char[]]$pPattern){
         switch ($chr){
             U {$thePass += get-random -InputObject $aUpper; break}
             L {$thePass += get-random -InputObject $aLower; break}
             N {$thePass += get-random -InputObject $aNumber; break}
             S {$thePass += get-random -InputObject $aSCharacter; break}
         }
     }
     $thePass
 }
 
 for ($i=0; $i -le 20; $i++){Get-RandomPassword -length 12}
`

