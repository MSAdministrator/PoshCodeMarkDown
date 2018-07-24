---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2210
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# search-startmenu.ps1 - 

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
 
 Search the Start Menu for items that match the provided text. This script
 searches both the name (as displayed on the Start Menu itself,) and the
 destination of the link.
 
 .Example
 
 Search-StartMenu "Character Map" | Invoke-Item
 Searches for the "Character Map" appication, and then runs it
 
 Search-StartMenu PowerShell | Select-FilteredObject | Invoke-Item
 Searches for anything with "PowerShell" in the application name, lets you
 pick which one to launch, and then launches it.
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Pattern
 )
 
 Set-StrictMode -Version Latest
 
 $myStartMenu = [Environment]::GetFolderPath("StartMenu")
 $shell = New-Object -Com WScript.Shell
 $allStartMenu = $shell.SpecialFolders.Item("AllUsersStartMenu")
 
 $escapedMatch = [Regex]::Escape($pattern)
 
 dir $myStartMenu *.lnk -rec | ? { $_.Name -match "$escapedMatch" }
 dir $allStartMenu *.lnk -rec | ? { $_.Name -match "$escapedMatch" }
 
 dir $myStartMenu *.lnk -rec |
     Where-Object { $_ | Select-String "\\[^\\]*$escapedMatch\." -Quiet }
 dir $allStartMenu *.lnk -rec |
     Where-Object { $_ | Select-String "\\[^\\]*$escapedMatch\." -Quiet }
`

