---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5323
Published Date: 2015-07-23t20
Archived Date: 2015-01-31t20
---

# add to start of file - add-beginningcontent.ps1

## Description

this script recurses through the directory in the $path variable checks which files

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 #
 
 $path = 'E:\SkyDrive\Documents\Scripts\Powershell Scripts\Functions\'
 $Files = Get-ChildItem -Path $path -Recurse
 
 $Names = $Files|select name, FullName
 
 $Matchs = $files |
   where { $_ | Select-String -Pattern "###"  } |
 
 
       $Diffs = $Names  | Where-Object {$Matchs.Name -notcontains  $_.Name}
 
 
       foreach($Diff in $Diffs)
       {
       $ScriptName = $Diff.Name
       $Old = Get-Content $Diff.FullName
       $Header = "#############################################################################################
 #
 #
 #
 #
       "
       Set-Content -Path $FileName -Value $Header
       Add-Content -Path $FileName -Value $Old 
       Invoke-Item $FileName
       pause
       }
`

