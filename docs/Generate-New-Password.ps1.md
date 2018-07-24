---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1945
Published Date: 2011-07-03t14
Archived Date: 2016-06-10t06
---

# generate new password - 

## Description

a function which can be added to a script or $profile which generates password from 1 to whatever characters long with controlled levels of complexity specified by the user

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function global:GET-NewPassword($PasswordLength, $Complexity) {
 
 <#
 
 .SYNOPSIS 
 Generates a New password with varying length and Complexity, 
 
 .DESCRIPTION 
 Generate a New Password for a User.  Defaults to 8 Characters
 with Moderate Complexity.  Usage
 
 GET-NEWPASSWORD or
 
 GET-NEWPASSWORD $Length $Complexity
 
 Where $Length is an integer from 1 to as high as you want
 and $Complexity is an Integer from 1 to 4
 
 .EXAMPLE 
 Create New Password
 
 GET-NEWPASSWORD
 
 .EXAMPLE 
 Generate a Password of strictly Uppercase letters that is 9 letters long
 
 GET-NEWPASSWORD 9 1
 
 .EXAMPLE 
 Generate a Highly Complex password 5 letters long 
 
 GET-NEWPASSWORD 5
 
 .EXAMPLE 
 Create a new 8 Character Password of Uppercase/Lowercase and store
 as a Secure.String in Variable called $MYPASSWORD
 
 $MYPASSWORD=CONVERTTO-SECURESTRING (GET-NEWPASSWORD 8 2) -asplaintext -force
 
 .NOTES 
 The Complexity falls into the following setup for the Complexity level
 1 - Pure lowercase Ascii
 2 - Mix Uppercase and Lowercase Ascii
 3 - Ascii Upper/Lower with Numbers
 4 - Ascii Upper/Lower with Numbers and Punctuation
 
 #>
 
 #
 [int32[]]$ArrayofAscii=26,97,26,65,10,48,15,33
 
 If ($Complexity -eq $NULL) { $Complexity=3 }
 
 If ($PasswordLength -eq $NULL) {$PasswordLength=10}
 
 $NewPassword=$NULL
 
 
 Foreach ($counter in 1..$PasswordLength) {
 
 
 $pickSet=(GET-Random $complexity)*2
 
 
 $NewPassword=$NewPassword+[char]((get-random $ArrayOfAscii[$pickset])+$ArrayOfAscii[$pickset+1])
 }
 
 Return $NewPassword
 
 }
`

