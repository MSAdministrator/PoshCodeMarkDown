---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2223
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# show-colorizedcontent.ps - 

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
 
 Displays syntax highlighting, line numbering, and range highlighting for
 PowerShell scripts.
 
 .EXAMPLE
 
 PS >Show-ColorizedContent Invoke-MyScript.ps1
 
 001 | function Write-Greeting
 002 | {
 003 |     param($greeting)
 004 |     Write-Host "$greeting World"
 005 | }
 006 |
 007 | Write-Greeting "Hello"
 
 .EXAMPLE
 
 PS >Show-ColorizedContent Invoke-MyScript.ps1 -highlightRange (1..3+7)
 
 001 > function Write-Greeting
 002 > {
 003 >     param($greeting)
 004 |     Write-Host "$greeting World"
 005 | }
 006 |
 007 > Write-Greeting "Hello"
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Path,
 
     $HighlightRange = @(),
 
     [Switch] $ExcludeLineNumbers
 )
 
 Set-StrictMode -Version Latest
 
 $replacementColours = @{
     'Attribute' = 'DarkCyan'
     'Command' = 'Blue'
     'CommandArgument' = 'Magenta'
     'CommandParameter' = 'DarkBlue'
     'Comment' = 'DarkGreen'
     'GroupEnd' = 'Black'
     'GroupStart' = 'Black'
     'Keyword' = 'DarkBlue'
     'LineContinuation' = 'Black'
     'LoopLabel' = 'DarkBlue'
     'Member' = 'Black'
     'NewLine' = 'Black'
     'Number' = 'Magenta'
     'Operator' = 'DarkGray'
     'Position' = 'Black'
     'StatementSeparator' = 'Black'
     'String' = 'DarkRed'
     'Type' = 'DarkCyan'
     'Unknown' = 'Black'
     'Variable' = 'Red'
 }
 
 $highlightColor = "Red"
 $highlightCharacter = ">"
 $highlightWidth = 6
 if($excludeLineNumbers) { $highlightWidth = 0 }
 
 $file = (Resolve-Path $Path).Path
 $content = [IO.File]::ReadAllText($file)
 $parsed = [System.Management.Automation.PsParser]::Tokenize(
     $content, [ref] $null) | Sort StartLine,StartColumn
 
 function WriteFormattedLine($formatString, [int] $line)
 {
     if($excludeLineNumbers) { return }
 
     $hColor = "DarkGray"
     $separator = "|"
 
     if($highlightRange -contains $line)
     {
         $hColor = $highlightColor
         $separator = $highlightCharacter
     }
 
     $text = $formatString -f $line,$separator
     Write-Host -NoNewLine -Fore $hColor -Back White $text
 }
 
 function CompleteLine($column)
 {
     $lineRemaining = $host.UI.RawUI.WindowSize.Width -
         $column - $highlightWidth + 1
 
     if($lineRemaining -lt 0)
     {
         $lineRemaining += $host.UI.RawUI.WindowSize.Width
     }
 
     Write-Host -NoNewLine -Back White (" " * $lineRemaining)
 }
 
 
 Write-Host
 WriteFormattedLine "{0:D3} {1} " 1
 
 $column = 1
 foreach($token in $parsed)
 {
     $color = "Gray"
 
     $color = $replacementColours[[string]$token.Type]
     if(-not $color) { $color = "Gray" }
 
     if(($token.Type -eq "NewLine") -or ($token.Type -eq "LineContinuation"))
     {
         CompleteLine $column
         WriteFormattedLine "{0:D3} {1} " ($token.StartLine + 1)
         $column = 1
     }
     else
     {
         if($column -lt $token.StartColumn)
         {
             $text = " " * ($token.StartColumn - $column)
             Write-Host -Back White -NoNewLine $text
             $column = $token.StartColumn
         }
 
         $tokenEnd = $token.Start + $token.Length - 1
 
         if(
             (($token.Type -eq "String") -or
             ($token.Type -eq "Comment")) -and
             ($token.EndLine -gt $token.StartLine))
         {
             $lineCounter = $token.StartLine
 
             $stringLines = $(
                 -join $content[$token.Start..$tokenEnd] -split "`n")
 
             foreach($stringLine in $stringLines)
             {
                 $stringLine = $stringLine.Trim()
 
                 if($lineCounter -gt $token.StartLine)
                 {
                     CompleteLine $column
                     WriteFormattedLine "{0:D3} {1} " $lineCounter
                     $column = 1
                 }
 
                 Write-Host -NoNewLine -Fore $color -Back White $stringLine
                 $column += $stringLine.Length
                 $lineCounter++
             }
         }
         else
         {
             $text = (-join $content[$token.Start..$tokenEnd])
             Write-Host -NoNewLine -Fore $color -Back White $text
         }
 
         $column = $token.EndColumn
     }
 }
 
 CompleteLine $column
 Write-Host
`

