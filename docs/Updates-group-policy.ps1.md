---
Author: johnny reel
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4722
Published Date: 2013-12-19t19
Archived Date: 2013-12-22t06
---

# updates group policy - 

## Description

updates group policy on remote domain computer,(can be modified easily to include all computers or a list.). i wrote this for our field techs, simple but useful.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 Import-Module -Name ActiveDirectory
 
 
 $cn = Get-ADComputer -Filter 'Name -like "<computer>"'
 $cred = $User
 $session = New-PSSession -ComputerName $cn.Name -Credential $cred
 Invoke-Command -Session $session -ScriptBlock {gpupdate /force}
`

