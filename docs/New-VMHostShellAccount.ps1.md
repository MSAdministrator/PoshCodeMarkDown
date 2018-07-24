---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 563
Published Date: 
Archived Date: 2008-09-15t21
---

# new-vmhostshellaccount - 

## Description

the vi toolkit comes with a cmdlet to create user accounts, but it does not allow for you to specify shell access. this script goes to the vi sdk to get the job done.

## Comments



## Usage



## TODO



## function

`new-vmhostshellaccount`

## Code

`#
 #
 function New-VMHostShellAccount {
 	param (
 		$Name,
 		$Password = $null, 
 		$Description = $null, 
 		$PosixId = $null
 	)
 	$SvcInstance = Get-View serviceinstance
 	$AcctMgr = Get-View $SvcInstance.Content.AccountManager
 	$AcctSpec = new-object VMware.Vim.HostPosixAccountSpec
 	$AcctSpec.id = $Name
 	$AcctSpec.password = $Password
 	$AcctSpec.description = $Description
 	$AcctSpec.posixId = $PosixId
 	Get-VMHostAccount | Where-Object { $_.Id -eq $Name }
 }
`

