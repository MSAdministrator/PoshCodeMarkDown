---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 859
Published Date: 2009-02-10t14
Archived Date: 2016-03-02t15
---

# ctp3 - 

## Description

requires ps 2.0 ctp3

## Comments



## Usage



## TODO



## function

`watch-folder`

## Code

`#
 #
 
 #
 
 function watch-folder {
     param([string]$folder)
     
     $fsw = new-object System.IO.FileSystemWatcher
     $fsw.Path = $folder
     
     $global:table = new-object system.data.datatable
     [void] $table.Columns.Add("FullPath", [string])
     [void] $table.Columns.Add("ChangeType", [string])
     
     $action = {
         [console]::beep(440,10)
         [void] $table.Rows.Add($eventArgs.FullPath, $eventArgs.ChangeType)
     }
         
     [void] Register-ObjectEvent -InputObject $fsw -EventName Created -Action $action
     [void] Register-ObjectEvent -InputObject $fsw -EventName Changed -Action $action
     [void] Register-ObjectEvent -InputObject $fsw -EventName Deleted -Action $action
 }
`

