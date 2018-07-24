---
Author: adam liquorish
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3107
Published Date: 2012-12-19t18
Archived Date: 2012-01-14t07
---

# user logon details - 

## Description

script to print to a html formatted table user details for a workstation.  user details include accountlockouttime,enabled,lastlogon,badlogoncount,lastpasswordset,lastbadpasswordattempt,passwordnotrequired,passwordneverexpires,usercannotchangepassword,allowreversiblepasswordencryption.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 ########
 #
 ########
 
 param(
 [Parameter(Mandatory=$true,
 	HelpMessage="Enter Path for output ie c:\Temp\output.html.")]
 	[ValidateNotNullOrEmpty()]
 	[string]$outputpath=$(Read-Host -prompt "Path for Output"),
 [Parameter(Mandatory=$true,
 	HelpMessage="Enter ComputerName, enter 'localhost' for current workstation")]
 	[ValidateNotNullOrEmpty()]
 	[string]$computername=$(Read-Host -prompt "Computername")
 )
 
 if($computername -eq "localhost"){$computername="$env:computername"}
 $i=$null
 $results=$null
 $useroutput=$null
 
 $a="<style>"
 $a=$a +"TABLE{border-width: 1px;border-style: solid;border-color: black;border-collapse: collapse;}"
 $a=$a +"TH{border-width: 1px;padding: 0px;border-style: solid;border-color: black;background-color: thistle}"
 $a=$a + "TD{border-width: 1px;padding: 0px;border-style: solid;border-color: black;}"
 $a=$a + "</style>"
 
 $computer=[ADSI]"WinNT://$computerName,computer"
 $users=$computer.psbase.Children|Where-object{$_.psbase.schemaclassname -eq 'user'}
 
 Add-Type -AssemblyName System.DirectoryServices.AccountManagement
 $domain= "$env:computername"
 $ctype = [System.DirectoryServices.AccountManagement.ContextType]::Machine
 $principal=new-object System.DirectoryServices.AccountManagement.PrincipalContext $ctype,$domain
 $useroutput=foreach($user in $users){
 	[System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($principal,$user.name)}
 	
 $results=$useroutput|select-object name,AccountLockoutTime,Enabled,LastLogon,BadLogonCount,LastPasswordSet,LastBadPasswordAttempt,PasswordNotRequired,PasswordNeverExpires,UserCannotChangePassword,AllowReversiblePasswordEncryption
 $results|sort-object Name|ConvertTo-Html -head $a -title "Local Users" -body "Local Users"|Set-Content $outputpath
`

