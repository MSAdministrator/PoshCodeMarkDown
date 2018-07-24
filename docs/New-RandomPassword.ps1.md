---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6273
Published Date: 2016-03-30t13
Archived Date: 2016-04-01t22
---

# new-randompassword - 

## Description

generates a random password of a specified length.

## Comments



## Usage



## TODO



## function

`new-randompassword`

## Code

`#
 #
 Function New-RandomPassword {
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
`

