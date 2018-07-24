---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2127
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t10
---

# watch-expression.ps1 - 

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
 
 Updates your prompt to display the values of information you want to track.
 
 
 .EXAMPLE
 
 PS >Watch-Expression { (Get-History).Count }
 
 Expression          Value
 ----------          -----
 (Get-History).Count     3
 
 PS >Watch-Expression { $count }
 
 Expression          Value
 ----------          -----
 (Get-History).Count     4
 $count
 
 PS >$count = 100
 
 Expression          Value
 ----------          -----
 (Get-History).Count     5
 $count                100
 
 PS >Watch-Expression -Reset
 PS >
 
 #>
 
 param(
     [ScriptBlock] $ScriptBlock,
 
     [Switch] $Reset
 )
 
 Set-StrictMode -Version Latest
 
 if($Reset)
 {
     Set-Item function:\prompt ([ScriptBlock]::Create($oldPrompt))
 
     Remove-Item variable:\expressionWatch
     Remove-Item variable:\oldPrompt
 
     return
 }
 
 if(-not (Test-Path variable:\expressionWatch))
 {
     $GLOBAL:expressionWatch = @()
 }
 
 $GLOBAL:expressionWatch += $scriptBlock
 
 $GLOBAL:oldPrompt = Get-Content function:\prompt
 if($oldPrompt -notlike '*$expressionWatch*')
 {
     $newPrompt = @'
         $results = foreach($expression in $expressionWatch)
         {
             New-Object PSObject -Property @{
                 Expression = $expression.ToString().Trim();
                 Value = & $expression
             } | Select Expression,Value
         }
         Write-Host "`n"
         Write-Host ($results | Format-Table -Auto | Out-String).Trim()
         Write-Host "`n"
 
 '@
 
     $newPrompt += $oldPrompt
 
     Set-Item function:\prompt ([ScriptBlock]::Create($newPrompt))
 }
`

