---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2164
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t00
---

# get-scriptcoverage.ps1 - 

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
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Uses conditional breakpoints to obtain information about what regions of
 a script are executed when run.
 
 .EXAMPLE
 
 PS >Get-Content c:\temp\looper.ps1
 
 param($userInput)
 
 for($count = 0; $count -lt 10; $count++)
 {
     "Count is: $count"
 }
 
 if($userInput -eq "One")
 {
     "Got 'One'"
 }
 
 if($userInput -eq "Two")
 {
     "Got 'Two'"
 }
 
 PS >$action = { c:\temp\looper.ps1 -UserInput 'One' }
 PS >$coverage = Get-ScriptCoverage c:\temp\looper.ps1 -Action $action
 PS >$coverage | Select Content,StartLine,StartColumn | Format-Table -Auto
 
 Content   StartLine StartColumn
 -------   --------- -----------
 param             1           1
 (                 1           6
 userInput         1           7
 )                 1          17
 Got 'Two'        15           5
 }                16           1
 
 This example exercises a 'looper.ps1' script, and supplies it with some
 user input. The output demonstrates that we didn't exercise the
 "Got 'Two'" statement.
 
 #>
 
 param(
     $Path,
 
     [ScriptBlock] $Action = { & $path }
 )
 
 Set-StrictMode -Version Latest
 
 $scriptContent = Get-Content $path
 $ignoreTokens = "Comment","NewLine"
 $tokens = [System.Management.Automation.PsParser]::Tokenize(
     $scriptContent, [ref] $null) |
     Where-Object { $ignoreTokens -notcontains $_.Type }
 $tokens = $tokens | Sort-Object StartLine,StartColumn
 
 $visited = New-Object System.Collections.ArrayList
 
 $breakpoints = foreach($token in $tokens)
 {
     $breakAction = { $null = $visited.Add($token) }.GetNewClosure()
 
     Set-PsBreakpoint $path -Line `
         $token.StartLine -Column $token.StartColumn -Action $breakAction
 }
 
 $null = . $action
 
 $breakpoints | Remove-PsBreakpoint
 
 $visited = $visited | Sort-Object -Unique StartLine,StartColumn
 Compare-Object $tokens $visited -Property StartLine,StartColumn -PassThru
 
 Remove-Item variable:\visited
`

