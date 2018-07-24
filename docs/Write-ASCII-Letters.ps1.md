---
Author: joakim svendsen, svendsen tech
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6671
Published Date: 2017-01-07t04
Archived Date: 2017-01-10t08
---

# write-ascii-letters - 

## Description

i initially wrote this ascii art character script to be used with a modified version of powerbot 2.0 (id 2510 on poshcode). it outputs ascii art letters from what you supply as a parameter (if the characters are supported). either to stdout with write-host (colors are supported) or to the pipeline. it’s useless without the xml that’s found at powershelladmin.com. the full article is in my wiki

## Comments



## Usage



## TODO



## script

`parse-letterxml`

## Code

`#
 #
 #
 #.SYNOPSIS
 #
 #
 #.DESCRIPTION
 #
 #
 #
 #
 #.PARAMETER InputText
 #.PARAMETER NoPrependChar
 #.PARAMETER TextColor
 #>
 
 param(
     [string[]] $InputText,
     [switch] $NoPrependChar,
     [string] $TextColor = 'Default'
     #[int] $MaxChars = '25'  
     )
 
 Set-StrictMode -Version Latest
 $ErrorActionPreference = 'Stop'
 
 Function Parse-LetterXML {
     
     $LetterFile = '.\letters.xml'
     $Xml = [xml] (Get-Content $LetterFile)
     
     $Xml.Letters.Letter | ForEach-Object {
         
         $Letters.($_.Name) = New-Object PSObject -Property @{
             
             'Lines' = $_.Lines
             'ASCII' = $_.Data
             'Width' = $_.Width
         }
         
     }
     
 }
 
 function Create-ASCII-Text {
     
     param([string] $Text)
     
     $LetterArray = [char[]] $Text.ToLower()
     
     
     $MaxLines = 0
     $LetterArray | ForEach-Object { if ($Letters.([string] $_).Lines -gt $MaxLines ) { $MaxLines = $Letters.([string] $_).Lines } }
     
     $LetterWidthArray = $LetterArray | ForEach-Object { $Letter = [string] $_; $Letters.$Letter.Width }
     $LetterLinesArray = $LetterArray | ForEach-Object { $Letter = [string] $_; $Letters.$Letter.Lines }
     
     #$LetterLinesArray
     
     $Lines = @{
         '1' = ''
         '2' = ''
         '3' = ''
         '4' = ''
         '5' = ''
         '6' = ''
     }
     
     #$LineLengths = @(0, 0, 0, 0, 0, 0)
         
     $LetterPos = 0
     foreach ($Letter in $LetterArray) {
         
         $Letter = [string] $Letter
         
         
         if ($LetterLinesArray[$LetterPos] -eq 6) {
             
             foreach ($Num in 1..6) {
                 
                 $StringNum = [string] $Num
                 
                 $LineFragment = [string](($Letters.$Letter.ASCII).Split("`n"))[$Num-1]
                 
                 if ($LineFragment.Length -lt $Letters.$Letter.Width) {
                     $LineFragment += ' ' * ($Letters.$Letter.Width - $LineFragment.Length)
                 }
                 
                 $Lines.$StringNum += $LineFragment
                 
             }
             
         }
         
         elseif ($LetterLinesArray[$LetterPos] -eq 5) {
             
             $Padding = ' ' * $LetterWidthArray[$LetterPos]
             $Lines.'1' += $Padding
             
             foreach ($Num in 2..6) {
                 
                 $StringNum = [string] $Num
                 
                 $LineFragment = [string](($Letters.$Letter.ASCII).Split("`n"))[$Num-2]
                 
                 if ($LineFragment.Length -lt $Letters.$Letter.Width) {
                     $LineFragment += ' ' * ($Letters.$Letter.Width - $LineFragment.Length)
                 }
                     
                 $Lines.$StringNum += $LineFragment
                 
             }
         
             
         }
         
         else {
             
             $StartRange, $EndRange, $IndexSubtract = 3, 6, 3
             $Padding = ' ' * $LetterWidthArray[$LetterPos]
             
             if ($MaxLines -lt 6) {
                 
                 $Lines.'2' += $Padding
                 
             }
            
             else {
                 
                 $Lines.'1' += $Padding
                 $Lines.'6' += $Padding
                 $StartRange, $EndRange, $IndexSubtract = 2, 5, 2
                 
             }
             
             foreach ($Num in $StartRange..$EndRange) {
                 
                 $StringNum = [string] $Num
                 
                 $LineFragment = [string](($Letters.$Letter.ASCII).Split("`n"))[$Num-$IndexSubtract]
                
                 if ($LineFragment.Length -lt $Letters.$Letter.Width) {
                     $LineFragment += ' ' * ($Letters.$Letter.Width - $LineFragment.Length)
                 }
                 
                 $Lines.$StringNum += $LineFragment
                 
             }
                 
         }
                     
         $LetterPos++
         
     
     $Lines.GetEnumerator() | Sort Name | Select -ExpandProperty Value | ?{ $_ -match '\S' } | %{ if ($NoPrependChar) { $_ } else { "'" + $_ } }
     
 }
 
 $Letters = @{}
 Parse-LetterXML
 
 $Text = ''
 $InputText | ForEach-Object { $Text += "$_ " }
 
 $MaxChars = 30
 if ($Text.Length -gt $MaxChars) { "Too long text. There's a maximum of $MaxChars characters."; exit }
 
 $Text = $Text -replace ' ', '_'
 
 if ($Text -match $AcceptedChars) { "Unsupported character, using this 'accepted chars' regex: $AcceptedChars."; exit }
 
 
 $Lines = @()
 $ASCII = Create-ASCII-Text $Text
 
 if ($TextColor -ne 'Default') { Write-Host -ForegroundColor $TextColor ($ASCII -join "`n") }
 else { $ASCII }
`

