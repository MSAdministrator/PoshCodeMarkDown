---
Author: nathan hartley
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 576
Published Date: 2009-09-10t07
Archived Date: 2016-06-21t10
---

# set-localpassword - 

## Description

get-oucomputernames

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 param(
 	[switch]$Help
 	, [string] $User
 	, [string] $Password
     , [string[]] $ComputerNames = @()
 )
 $usage = @'
 Get-OUComputerNames
 usage   : [computerName1,computerName2,... | ] ./Set-LocalPassword.ps1 [-user] <userName> [-password] <password> [[-computers] computerName1,computerName2,...]
 returns : Sets local account passwords on one or more computers
 author  : Nathan Hartley
 '@
 if ($help) {Write-Host $usage;break}
 
 $ComputerNames += @($input)
 
 if (! $ComputerNames)
 {
     $ComputerNames = $env:computername
 }
 
 function ChangePassword ([string] $ChangeComputer, [string] $ChangeUser, [string] $ChangePassword) {
 	"*** Setting password for $ChangeComputer/$ChangeUser"
 	& {
 	$ErrorActionPreference="silentlycontinue"
 	([ADSI] "WinNT://$computer/$user").SetPassword($password)
 	if ($?) { " Success" }
 	else { " Failed: $($error[0])" }
 	}
 
 }
 
 ForEach ($computer in $ComputerNames) { 
 	ChangePassword $computer $user $password 
 }
`

