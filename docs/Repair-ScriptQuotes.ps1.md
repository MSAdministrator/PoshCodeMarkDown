---
Author: p sczepanski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3816
Published Date: 2012-12-11t13
Archived Date: 2012-12-16t04
---

# repair-scriptquotes - 

## Description

script to replace special quote characters when copy – pasting from word or from blogs.

## Comments



## Usage



## TODO



## function

`repair-scriptquotes`

## Code

`#
 #
 function Repair-ScriptQuotes {
     param (
         [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
         [string]$path
     )
     if ( !( Test-Path $path ) ) {
         Write-Warning "'$path' does not exist."
         Continue
     }
         
 
     (( Get-Content $path) |  
         ForEach-Object {
             [Net.WebUtility]::HtmlDecode($_)
         } | 
             ForEach-Object {
                 $_ -replace "(\u201c|\u201d|\u201e)",[char]34
             } | 
                 ForEach-Object {
                     $_ -replace "(\u2018|\u2019|\u201a)",[char]39
                 } | 
                     ForEach-Object {
                         $_ -replace "(\u2013)",[char]45
                     } | 
                         ForEach-Object {
                             $_ -replace "`t",(" " * 4)
                         }) | 
                             Set-Content $path
     Write-Host "Done"
 }
`

