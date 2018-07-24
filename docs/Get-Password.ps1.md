---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4532
Published Date: 2013-10-19t11
Archived Date: 2013-10-25t00
---

# get-password - 

## Description

another password generating function. this one will always include all the character types you want, and the password will always be random.

## Comments



## Usage



## TODO



## function

`get-password`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Get-Password
 {
     param(
     [int] $PasswordLength = 10)
 
     $NrOfCharacterTypes=4
 
 
     if ($PasswordLength -lt $NrOfCharacterTypes) {
         Write-Error "The password must be at least $NrOfCharacterTypes characters long."
         return
     }
 
     $CapitalLetters=For ($a=65;$a �le 90;$a++) { [char][byte]$a }
     $SmallLetters=For ($a=97;$a �le 122;$a++) { [char][byte]$a }
     $Numbers=0..9
     $SpecialCharacters="!","@","#","%","&","?","+","*","=","£","\"
 
     $AllCharacters=$CapitalLetters+$SmallLetters+$Numbers+$SpecialCharacters
 
     for ($CharNr=$NrOfCharacterTypes;$CharNr -le $PasswordLength;$CharNr++) {
         
         if ($CharNr -eq $NrOfCharacterTypes) {
             $CharacterString+=($CapitalLetters | Get-Random)
             $CharacterString+=($SmallLetters | Get-Random)
             $CharacterString+=($Numbers | Get-Random)
             $CharacterString+=($SpecialCharacters | Get-Random)
         }
         else {
             $CharacterString+=($AllCharacters | Get-Random)
         }
     }
 
     $TempPasswordArray=$CharacterString.ToString().ToCharArray()
 
     $PasswordToReturn=($TempPasswordArray | Get-Random -Count $TempPasswordArray.length) -join ''
 
     Write-Output $PasswordToReturn
 }
`

