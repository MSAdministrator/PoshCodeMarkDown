---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2160
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# get-parameteralias.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

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
 
 Looks in the session history, and returns any aliases that apply to
 parameters of commands that were used.
 
 .EXAMPLE
 
 PS >dir -ErrorAction SilentlyContinue
 PS >Get-ParameterAlias
 An alias for the 'ErrorAction' parameter of 'dir' is ea
 
 #>
 
 Set-StrictMode -Version Latest
 
 $history = Get-History -Count 1
 if(-not $history)
 {
     return
 }
 
 $lastCommand = $history.CommandLine
 
 $tokens = [System.Management.Automation.PsParser]::Tokenize(
     $lastCommand, [ref] $null)
 $currentCommand = $null
 
 foreach($token in $tokens)
 {
     if($token.Type -eq "Command")
     {
         $currentCommand = $token.Content
     }
 
     if(($token.Type -eq "CommandParameter") -and ($currentCommand))
     {
         $currentParameter = $token.Content.TrimStart("-")
 
         (Get-Command $currentCommand).Parameters.GetEnumerator() |
 
             Where-Object { $_.Key -like "$currentParameter*" } |
 
             Foreach-Object {
                 $_.Value.Aliases | Foreach-Object {
                     "Suggestion: An alias for the '$currentParameter' " +
                     "parameter of '$currentCommand' is '$_'"
                 }
             }
     }
 }
`

