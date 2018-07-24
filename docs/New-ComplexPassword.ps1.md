---
Author: jon webster
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 567
Published Date: 2009-09-05t20
Archived Date: 2012-06-16t07
---

# new-complexpassword - 

## Description

generate random passwords with specific complexity requirements

## Comments



## Usage



## TODO



## function

`new-complexpassword`

## Code

`#
 #
 
 Function New-ComplexPassword ([int]$Length=8, $digits=$null, $alphaUpper=$null, $alphaLower=$null, $special=$null)
 {
 
 
 	if($digits+$alphaUpper+$alphaLower+$special -gt $Length) { throw "Password too short for specified complexity" }
 
 
 	
 	$groups = @()
 	$group = New-Object System.Object
 
 	Add-Member -In $group -Type NoteProperty -Name "Count" -Value $Digits
 	$groups += $group
 
 	$group = New-Object System.Object
 	Add-Member -In $group -Type NoteProperty -Name "Count" -Value $alphaUpper
 	$groups += $group
 
 	$group = New-Object System.Object
 	Add-Member -In $group -Type NoteProperty -Name "Count" -Value $alphaLower
 	$groups += $group
 
 	$group = New-Object System.Object
 	Add-Member -In $group -Type NoteProperty -Name "Count" -Value $special
 	$groups += $group 
 
 	$ran = New-Object Random
 
 	foreach ($req in $groups)
 	{
 		if ($req.count)
 		{
 			$charsAllowed += $req.group
 
 			for ($i=0; $i -lt $req.count; $i++)
 			{
 				$r = $ran.Next(0,$req.group.length)
 				$password += $req.group[$r]	
 			}
 		} elseif ($req.count -eq 0) {
 			$charsAllowed += $req.group
 		}
 	}
 
 	if(!$charsAllowed)
 	{
 		$groups |% { $charsAllowed += $_.group }
 	}
 
 	for($i=$password.length; $i -lt $length; $i++)
 	{
 		$r = $ran.Next(0,$charsAllowed.length)
 		$password += $charsAllowed[$r]
 	}
 
 	return [string]::join('',($password.ToCharArray()|sort {$ran.next()}))
 }
`

