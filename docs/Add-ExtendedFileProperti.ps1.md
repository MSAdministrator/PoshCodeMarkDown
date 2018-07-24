---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5373
Published Date: 2016-08-19t22
Archived Date: 2016-03-18t21
---

# add-extendedfileproperti - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Add the extended file properties normally shown in Exlorer's
 "File Properties" tab.
 
 .EXAMPLE
 
 Get-ChildItem | Add-ExtendedFileProperties.ps1 | Format-Table Name,"Bit Rate"
 
 #>
 
 begin
 {
     Set-StrictMode -Version Latest
 
     $shellObject = New-Object -Com Shell.Application
 
     $itemProperties = $null
 }
 
 process
 {
     $fileItem = $_ | Get-Item
 
     if($fileItem.PsIsContainer)
     {
         $fileItem
         return
     }
 
     $directoryName = $fileItem.DirectoryName
     $filename = $fileItem.Name
 
     $folderObject = $shellObject.NameSpace($directoryName)
     $item = $folderObject.ParseName($filename)
 
     if(-not $itemProperties)
     {
         $itemProperties = @{}
 
         $counter = 0
         $columnName = ""
         do
         {
             $columnName = $folderObject.GetDetailsOf(
                 $folderObject.Items, $counter)
             if($columnName) { $itemProperties[$counter] = $columnName }
 
             $counter++
         } while($columnName)
     }
 
     foreach($itemProperty in $itemProperties.Keys)
     {
         $fileItem | Add-Member NoteProperty $itemProperties[$itemProperty] `
             $folderObject.GetDetailsOf($item, $itemProperty) -ErrorAction `
             SilentlyContinue
     }
 
     $fileItem
 }
`

