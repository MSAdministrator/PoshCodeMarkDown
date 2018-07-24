---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 828
Published Date: 
Archived Date: 2009-02-04t17
---

# new-customcolumn () - 

## Description

new-customcolumn for powershell v1.0

## Comments



## Usage



## TODO



## function

`new-customcolumn`

## Code

`#
 #
 #
 #
 
 Function New-CustomColumn {
   PARAM (
     $Expression,
     $name,
     $label,
     $FormatString,
     [int]$Width,
     $Alignment
   )
 
   $column = @{}
 
   if ($Expression){
     $Column.Expression = $Expression
   } else {
     throw "Expression is mandatory"
   }
   if ($Name) {$Column.Name = $name}
   if ($Label) {$Column.Label = $Label}
   if ($FormatString) {$Column.FormatString = $FormatString}
   if ($Width) {$Column.Width = $Width}
   if ($Alignment) {$Column.Alignment = $Alignment}
 
   $Column.psobject.baseobject
 
 }
`

