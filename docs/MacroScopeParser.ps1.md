---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1733
Published Date: 
Archived Date: 2010-04-09t13
---

# macroscopeparser - 

## Description

uses macroscope/antlr to parse sql query for tables and table aliases

## Comments



## Usage



## TODO



## function

`get-table`

## Code

`#
 #
 
 
 param ($commandText)
 
 add-type -Path $(Resolve-Path .\Antlr3.Runtime.dll | Select-Object -ExpandProperty Path)
 add-type -Path $(Resolve-Path .\MacroScope.dll | Select-Object -ExpandProperty Path)
 
 #######################
 function Get-Table
 {
     param($table)
 
     $table
 
     if ($table.HasNext)
     { Get-Table $table.Next }
     
 }
 
 $sqlparser =[MacroScope.Factory]::CreateParser($commandText)
 $expression = $sqlparser.queryExpression()
 Get-Table $expression.From.Item | Select @{n='Name';e={$_.Source.Identifier}}, @{n='Alias';e={$_.Alias}}
`

