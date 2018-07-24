---
Author: alexander petrovskiy                                                                              #
Publisher: alexander petrovskiy, softwaretestingusingpowershell.wordpress.com                                #
Copyright: © 2011 alexander petrovskiy, softwaretestingusingpowershell.wordpress.com. all rights reserved.   #
Email: 
Version: 4.1
Encoding: utf-8
License: cc0
PoshCode ID: 2914
Published Date: 2011-08-14t08
Archived Date: 2011-08-25t00
---

# esximgmt module sample 5 - 

## Description

this sample register all the virtual machines laying as folders on a esxi host.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #######################################################################################################################
 #######################################################################################################################
 param([string]$Server,
 	  [string]$User,
 	  [string]$Password,
 	  [string]$DatastoreName,
 	  [string]$Drive
 	  )
 
 cls
 Set-StrictMode -Version Latest
 Import-Module ESXiMgmt -Force;
 
 Connect-ESXi -Server $Server -Port 443 `
 	-Protocol HTTPS -User $User -Password $Password `
 	-DatastoreName $DatastoreName -Drive $Drive;
 
 dir "$($Drive):" | %{ `
 		if (Test-Path "$($_.FullName)\$($_.Name).vmx")
 		{
 			Register-ESXiVM  -Server $Server `
 				-User $User -Password $Password `
 				-Path "/vmfs/volumes/$($DatastoreName)/$($_.Name)/$($_.Name).vmx" `
 				-OperationTImeout 5;
 		}
 	}
`

