---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2129
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t10
---

# add-formattableindexpara - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

`add-indexparameter`

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
 
 Adds a new -IncludeIndex switch parameter to the Format-Table command
 to help with array indexing.
 
 .NOTES
 
 This commands builds on New-CommandWrapper, also included in the Windows
 PowerShell Cookbook.
 
 .EXAMPLE
 
 PS >$items = dir
 PS >$items | Format-Table -IncludeIndex
 PS >$items[4]
 
 #>
 
 Set-StrictMode -Version Latest
 
 New-CommandWrapper Format-Table `
     -AddParameter @{
         @{
             Name = 'IncludeIndex';
             Attributes = "[Switch]"
         } = {
 
         function Add-IndexParameter {
             begin
             {
                 $psIndex = 0
             }
             process
             {
                 if($_.GetType().FullName -eq `
                     "Microsoft.PowerShell.Commands.Internal." +
                     "Format.FormatStartData")
                 {
                     $formatStartType =
                         $_.shapeInfo.tableColumnInfoList[0].GetType()
                     $clone =
                         $formatStartType.GetConstructors()[0].Invoke($null)
 
                     $clone.PropertyName = "PSIndex"
                     $clone.Width = $clone.PropertyName.Length
 
                     $_.shapeInfo.tableColumnInfoList.Insert(0, $clone)
                 }
 
                 if($_.GetType().FullName -eq `
                     "Microsoft.PowerShell.Commands.Internal." +
                     "Format.FormatEntryData")
                 {
                     $firstField =
                         $_.formatEntryInfo.formatPropertyFieldList[0]
                     $formatFieldType = $firstField.GetType()
                     $clone =
                         $formatFieldType.GetConstructors()[0].Invoke($null)
 
                     $clone.PropertyValue = $psIndex
                     $psIndex++
 
                     $_.formatEntryInfo.formatPropertyFieldList.Insert(
                         0, $clone)
                 }
 
                 $_
             }
         }
 
         $newPipeline = { __ORIGINAL_COMMAND__ | Add-IndexParameter }
     }
 }
`

