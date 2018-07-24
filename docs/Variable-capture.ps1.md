---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4694
Published Date: 2013-12-13t00
Archived Date: 2013-12-16t12
---

# variable capture - 

## Description

powershell really needs lexical variables and automatic lexical closures. scriptblock.getnewclosure is a heavyweight hack (it captures the entire scope chain every time you call it) around lexical variable capture so here is a lighter weight hack.

## Comments



## Usage



## TODO



## function

`new-closure`

## Code

`#
 #
 function New-Closure {
 #.SYNOPSIS
 #.EXAMPLE
 	[OutputType([scriptblock])]
 	[CmdletBinding()]
 	param(
 		[Parameter(Mandatory)]
 		[System.Collections.IDictionary]$Variable
 ,
 		[Parameter(Mandatory)]
 		[scriptblock]$Script
 	)
 	try {
 		$private:m = New-Object System.Management.Automation.PSModuleInfo $true
 		$Script = $m.NewBoundScriptBlock($Script)
 		foreach ($v in $Variable.GetEnumerator()) {
 			& $m { Set-Variable -Name $args[0] -Value $args[1] -Scope script -Option AllScope } $v.Key $v.Value
 		}
 		$Script
 	} catch {
 		throw
 	}
 }
`

