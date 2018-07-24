---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6902
Published Date: 2017-05-22t17
Archived Date: 2017-05-30t19
---

# remote install - 

## Description

this script allows an administrator to install software from either a local folder on their administration pc or from a network share. target computers to receive the installation are defined ahead of time in a text file.

## Comments



## Usage



## TODO



## function

`install-software`

## Code

`#
 #
 function Install-Software{
 #.Synopsis
 #.Description
 #
 #.Parameter Targets
 #.Parameter Install
 #.Parameter Arguments
 #.Example
 #
 #.Example
 #
 
 [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
 param( 
 	[parameter(Mandatory = $true, Position = 0)] 
 	[string]$Targets,
 	[parameter(Mandatory = $true, Position = 1)] 
 	[string]$Install,
 	[parameter(Mandatory = $false, Position = 2)] 
 	[string]$Arguments
 ) 
 
 $Computers = Get-Content $Targets
 $InstallString = "$Install $Arguments"
 
 foreach ($Computer in $Computers) {
 	Copy-Item "$Install" \\$Computer\c$\TEMP
 	
 	Invoke-Command -ComputerName $Computer -ScriptBlock {
 		Start-Process "$InstallString"}
 }
`

