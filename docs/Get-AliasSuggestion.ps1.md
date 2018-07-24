---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2146
Published Date: 2010-09-09t21
Archived Date: 2016-12-02t04
---

# get-aliassuggestion.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Get an alias suggestion from the full text of the last command. Intended to
 be added to your prompt function to help learn aliases for commands.
 
 .EXAMPLE
 
 Get-AliasSuggestion Remove-ItemProperty
 Suggestion: An alias for Remove-ItemProperty is rp
 
 #>
 
 param(
     $LastCommand
 )
 
 Set-StrictMode -Version Latest
 
 $helpMatches = @()
 
 $tokens = [Management.Automation.PSParser]::Tokenize(
     $lastCommand, [ref] $null)
 $commands = $tokens | Where-Object { $_.Type -eq "Command" }
 
 foreach($command in $commands)
 {
     foreach($alias in Get-Alias -Definition $command.Content)
     {
     $helpMatches += "Suggestion: An alias for " +
         "$($alias.Definition) is $($alias.Name)"
     }
 }
 
 $helpMatches
`

