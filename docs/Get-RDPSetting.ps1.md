---
Author: jason archer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 678
Published Date: 
Archived Date: 2009-01-05t16
---

# get-rdpsetting - 

## Description

function/script to get the settings from a rdp file for terminal services.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ########################################################################################################################
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
     param(
         [string]$Path = $(throw "A path to a RDP file is required."),
         [string]$Name = "*"
     )
 
     $connection = Get-ChildItem -Path $path
 
     Get-Content -Path $Path | ForEach-Object {
         [Void] ($_ -match '^([^:]*):([^:]*):(.*)$')
         $settingname = $Matches[1]
         $type = $Matches[2]
         $value = $Matches[3]
         
         if ($settingname -like $Name) {
             switch ($type) {
                 'b' { $datatype = "byte[]" }
                 'i' { $datatype = "integer" }
                 default { $datatype = "string" }
             }
             
             $object = "" | Select-Object Name,DataType,Value,Connection
             $object.Name = $settingname
             $object.DataType = $datatype
             $object.Value = $value
             $object.Connection = $connection.FullName
             $object
         }
     }
 #}
`

