---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2176
Published Date: 2011-09-09t21
Archived Date: 2017-05-15t14
---

# invoke-cmdscript.ps1 - 

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
 
 Invoke the specified batch file (and parameters), but also propigate any
 environment variable changes back to the PowerShell environment that
 called it.
 
 .EXAMPLE
 
 PS >type foo-that-sets-the-FOO-env-variable.cmd
 @set FOO=%*
 echo FOO set to %FOO%.
 
 PS >$env:FOO
 PS >Invoke-CmdScript "foo-that-sets-the-FOO-env-variable.cmd" Test
 
 C:\Temp>echo FOO set to Test.
 FOO set to Test.
 
 PS > $env:FOO
 Test
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [string] $Path,
 
     [string] $ArgumentList
 )
 
 Set-StrictMode -Version Latest
 
 $tempFile = [IO.Path]::GetTempFileName()
 
 cmd /c " `"$Path`" $argumentList && set > `"$tempFile`" "
 
 Get-Content $tempFile | Foreach-Object {
     if($_ -match "^(.*?)=(.*)$")
     {
         Set-Content "env:\$($matches[1])" $matches[2]
     }
 }
 
 Remove-Item $tempFile
`

