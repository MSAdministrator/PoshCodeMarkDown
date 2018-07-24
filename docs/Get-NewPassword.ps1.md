---
Author: johnwcannon at gmail dot com
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2237
Published Date: 
Archived Date: 2010-09-22t05
---

# get-newpassword - 

## Description

password gerneration function originally posted on poshcode.org by sean kearney.  without any arguments, get-newpassword generates an 8 character password with upper, lower, and numeric characters.  it can accept 2 options, the first is password length (up to 94 chars) and the other is complexity (1 to 4).  i had to modify the code so that it wouldn’t create repeating characters.  the code may be ugly, but it’s because i am a network admin and not a programmer.  i hope it helps someone.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\
 #
 #
 #/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\/\
 
 function global:Get-NewPassword([int]$PasswordLength, [int]$Complexity) {
 
 if ($PasswordLength -ne 0)
 	{
 	if ($PasswordLength -gt 94)
 		{
 		Write-Host "That's a long password you requested!  There aren't enough unique characters to make a password that long."
 		Return
 		}
 	if ($PasswordLength -lt 1)
 		{
 		Write-Host $PasswordLength
 		Write-Host "You have entered an invalid password length."
 		Return
 		}
 	}
 if ($Complexity -ne 0)
 	{
 	if (($Complexity -lt 1) -or ($Complexity -gt 4))
 		{
 		Write-Host "You entered an invalid complexity value!"
 		Return
 		}
 	}
 If ($PasswordLength -eq 0)
 	{
 	$PasswordLength=8
 	}
 If ($Complexity -eq 0) { $Complexity=3 }
 
 if (($Complexity -eq 3) -and ($PasswordLength -gt 62))
 	{
 	Write-Host "You have requested a password that has more characters than the `ncomplexity you chose.  Try a complexity of 4 next time"
 	Return
 	}
 if (($Complexity -eq 2) -and ($PasswordLength -gt 52))
 	{
 	Write-Host "You have requested a password that has more characters than the `ncomplexity you chose.  Try a complexity of 3 or 4 next time"
 	Return
 	}
 if (($Complexity -eq 1) -and ($PasswordLength -gt 26))
 	{
 	Write-Host "You have requested a password that has more characters than the `ncomplexity you chose.  Try a complexity of 2, 3 or 4 next time"
 	Return
 	}
 
 $uppercasedec = 65..90
 $lowercasedec = 97..122
 $numeraldec = 48..57
 $specialchardec = 33..47
 $specialchardec1 = 58..64
 $specialchardec2 = 91..96
 $specialchardec3 = 123..126
 
 
 
 if($Complexity -eq 1)
 	{
 	$passchars += $lowercasedec
 	}
 if($Complexity -eq 2)
 	{
 	$passchars += $lowercasedec + $uppercasedec
 	}
 if($Complexity -eq 3)
 	{
 	$passchars += $lowercasedec + $uppercasedec + $numeraldec
 	}
 if($Complexity -eq 4)
 	{
 	$passchars += $lowercasedec + $uppercasedec + $numeraldec + `
 	$specialchardec + $specialchardec1 + $specialchardec2 + $specialchardec3 
 	}
 	
 
 Foreach ($counter in 1..$PasswordLength)
 	{
 	$lastchar = $null
 	$tempchar = ($passchars | Get-Random)
 	
 	$lastchar = $tempchar
 
 	If (($lastchar -eq ($tempchar + 1)) -or ($lastchar -eq ($tempchar - 1)) -and ($passchars.count -gt 2))
 		{
 		$tempchar = ($passchars | Get-Random)
 		
 		$lastchar = $tempchar
 		}
 	$passchars = ($passchars | Where-Object {$_ -ne $tempchar})
 	
 	$NewPassword=$NewPassword+[char]($tempchar)
 	}
 
 Return $NewPassword
 
 }
`

