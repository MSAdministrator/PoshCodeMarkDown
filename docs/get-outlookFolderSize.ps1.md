---
Author: marcadamcarter
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2641
Published Date: 2011-04-30t13
Archived Date: 2011-05-03t14
---

# get-outlookfoldersize - 

## Description

enumerate through the default folder within outlook and calculate size of each sub-folder and produce a final total size.

## Comments



## Usage



## TODO



## function

`get-folderitems`

## Code

`#
 #
 
 function get-folderItems
 {
     Param($parent)
     $parent.Items | % { $script:intSize += $_.Size }
     $currentSize = "{0:N2}" -f (($script:intSize - $script:prevSize)/1MB)
     $obj = New-Object PsObject
     $obj | Add-Member -memberType NoteProperty "Folder" -Value $parent.FolderPath
     $obj | Add-Member -memberType NoteProperty "Weight(MB)" -Value $currentSize
     $obj 
     $script:array += $obj
     $script:prevSize = $script:intSize
     
     foreach($folder in $parent.Folders)
     {
         get-folderItems $folder
     }
 }
 
 $array = @()
 $intSize = 0
 $prevSize = 0
 $o = new-object -comobject outlook.application 
 $n = $o.GetNamespace("MAPI")
 $objFolder = $n.GetDefaultFolder("olFolderInbox")
 get-folderItems $objFolder
 
 $obj = New-Object PsObject
 $obj | Add-Member -memberType NoteProperty "Folder" -Value "(Total)"
 $obj | Add-Member -memberType NoteProperty "Weight" -Value $("{0:N2}" -f ($intSize/1MB))
 $script:array += $obj
 $array | ft -auto
`

