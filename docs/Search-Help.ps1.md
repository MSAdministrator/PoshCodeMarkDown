---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2208
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# search-help.ps1 - 

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
 
 Search the PowerShell help documentation for a given keyword or regular
 expression.
 
 .EXAMPLE
 
 Search-Help hashtable
 Searches help for the term 'hashtable'
 
 .EXAMPLE
 
 Search-Help "(datetime|ticks)"
 Searches help for the term datetime or ticks, using the regular expression
 syntax.
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Pattern
 )
 
 Set-StrictMode -Version Latest
 
 $helpNames = $(Get-Help * | Where-Object { $_.Category -ne "Alias" })
 
 foreach($helpTopic in $helpNames)
 {
     $content = Get-Help -Full $helpTopic.Name | Out-String
     if($content -match "(.{0,30}$pattern.{0,30})")
     {
         $helpTopic | Add-Member NoteProperty Match $matches[0].Trim()
         $helpTopic | Select-Object Name,Match
     }
 }
`

