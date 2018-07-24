---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4109
Published Date: 2013-04-16t05
Archived Date: 2013-05-09t10
---

# get-comment - 

## Description

show all the comments from a script, and only the comments.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 <#
   .Synopsis
     Gets all of the comments from a script
   .Description
     Uses the PowerShell 3 Parser to figure out what's a comment
 #>
 [CmdletBinding()]
   [String]$Script 
 )
 
 if(Test-Path $Script) { 
 }
 $ParseError = $null
 $Tokens = $null
 $null = [System.Management.Automation.Language.Parser]::ParseInput($Script, [ref]$Tokens, [ref]$ParseError)
 $Tokens | ? Kind -eq "Comment" | % Text
`

