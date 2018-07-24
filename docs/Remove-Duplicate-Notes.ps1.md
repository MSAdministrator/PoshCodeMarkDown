---
Author: michaelj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3451
Published Date: 2012-06-10t19
Archived Date: 2012-06-18t09
---

# remove duplicate notes - 

## Description

remove duplicate notes from outlook.  i had many duplicate notes after a bad sync with my smartphone.  this removed them.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $outlook = new-object -comobject outlook.application
 
 $session = $outlook.session
 $session.logon()
 
 $olFoldersEnum = "Microsoft.Office.Interop.Outlook.OlDefaultFolders" -as [type]
 $notes = $session.getdefaultfolder($olFoldersEnum::olFolderNotes).items
 
 Write-Host "Number of notes:" $notes.count
 
 $noteStringModifiedDateMap = @{}
 $notesToDelete = @()
 
 Foreach ($e in $notes)
     { 
         $noteText = $e.Body
         $noteModified = $e.LastModificationTime
         if ($noteStringModifiedDateMap.ContainsKey($noteText))
         {
             $mapElement = $noteStringModifiedDateMap.Get_Item($noteText)
                 { $notesToDelete += $mapElement } 
             else
                 { $notesToDelete += $e }
         }
         else
         { $noteStringModifiedDateMap.Add($noteText, $e) }
      }
      
 $notesToDelete | Export-Csv "C:\toDelete.csv"
 
 foreach ($e in $notesToDelete)
 { $e.Delete() }
 
 Write-Host "done!"
`

