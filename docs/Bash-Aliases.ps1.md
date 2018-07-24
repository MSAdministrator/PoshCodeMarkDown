---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2330
Published Date: 
Archived Date: 2010-11-02t18
---

# bash aliases - 

## Description

this is an old script i had lying around. it has not been updated in a long time

## Comments



## Usage



## TODO



## module

`get-aliasfor`

## Code

`#
 #
 function alias {
    $alias,$cmd = [string]::join(" ",$args).split("=",2) | % { $_.trim()}
 
    if($Host.Version.Major -ge 2) {
       $cmd = Resolve-Aliases $cmd
    }
    New-Item -Path function: -Name "Global:Alias$Alias" -Options "AllScope" -Value @"
 Invoke-Expression '$cmd `$args'
 "@
 
    Set-Alias -Name $Alias -Value "Alias$Alias" -Description "A UNIX-style alias using functions" -Option "AllScope" -scope Global -passThru
 }
 
 function unalias([string]$Alias,[switch]$Force){ 
    if( (Get-Alias $Alias).Description -eq "A UNIX-style alias using functions" ) {
       Remove-Item "function:Alias$Alias" -Force:$Force
       Remove-Item "alias:$alias" -Force:$Force
       if($?) {
          "Removed alias '$Alias' and accompanying function"
       }
    } else {
       Remove-Item "alias:$alias" -Force:$Force
       if($?) {
          "Removed alias '$Alias'"
       }
    }
 }
 
 function Get-AliasFor([string]$CommandName) {
   ls Alias: | ?{ $_.Definition -match $CommandName }
 }
 
 Export-ModuleMember alias,unalias,Get-AliasFor
`

