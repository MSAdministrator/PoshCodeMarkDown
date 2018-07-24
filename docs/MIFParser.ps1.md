---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1447
Published Date: 2011-11-02t12
Archived Date: 2011-08-27t18
---

# mifparser.ps1 - 

## Description

parses management information format (mif) files. see http

## Comments



## Usage



## TODO



## function

`convertto-mifxml`

## Code

`#
 #
 param ($fileName, $computerName=$env:ComputerName)
 
 #######################
 function ConvertTo-MIFXml
 {
     param ($mifFile)
 
     $mifText = gc $mifFile | 
     % { $_ -replace "&", "&amp;" } |
     % { $_ -replace"'", "&apos;" } |
     % { $_ -replace "<", "&lt;" } |
     % { $_ -replace ">", "&gt;" } |
     % { $_ -replace 'Start Component','<Component' } | 
     % { $_ -replace 'Start Group','><Group' } | 
     % { $_ -replace 'Start Attribute','><Attribute' } |
     % { $_ -replace 'End Attribute','></Attribute>' } |
     % { $_ -replace 'End Group','</Group>' } |
     % { $_ -replace 'End Component','</Component>'} |
     % { $_ -replace '"' } | 
     % { $_ -replace "(\s*//\s*.*)" } |
     % { $_ -replace "\s*([^\s]+)\s*=\s*(.+$)",'$1="$2"' } |
     % { $_ -replace "\t", " " } |
     % { $_ -replace "\s{2,}", " " }
 
     [xml]$mifXml = [string]::Join(" ", $mifText) -replace ">\s*>",">" -replace "\s+>",">"
 
     return $mifXml
 
 
 #######################
 ConvertTo-MIFXml $fileName | foreach {$_.component} | foreach {$_.Group} | foreach {$_.Attribute} | select @{n='SystemName';e={$computerName}}, `
 @{n='Component';e={$($_.psbase.ParentNode).psbase.ParentNode.name}}, @{n='Group';e={$_.psbase.ParentNode.name}}, `
 @{n='FileName';e={[System.IO.Path]::GetFileName($FileName)}}, ID, Name, Value
`

