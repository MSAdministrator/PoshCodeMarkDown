---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3476
Published Date: 2013-06-25t06
Archived Date: 2016-03-18t21
---

# get-userlogonlogoffscrip - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Get the logon or logoff scripts assigned to a specific user
 
 .EXAMPLE
 
 Get-UserLogonLogoffScript LEE-DESK\LEE Logon
 Gets all logon scripts for the user 'LEE-DESK\Lee'
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Username,
 
     [Parameter(Mandatory = $true)]
     [ValidateSet("Logon","Logoff")]
     $ScriptType
 )
 
 Set-StrictMode -Version Latest
 
 $account = New-Object System.Security.Principal.NTAccount $username
 $sid =
     $account.Translate([System.Security.Principal.SecurityIdentifier]).Value
 
 $registryKey = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\" +
     "Group Policy\State\$sid\Scripts"
 
 if(-not (Test-Path $registryKey))
 {
     return
 }
 
 foreach($policy in Get-ChildItem $registryKey\$scriptType)
 {
     foreach($script in Get-ChildItem $policy.PsPath)
     {
         Get-ItemProperty $script.PsPath | Select Script,Parameters
     }
 }
`

