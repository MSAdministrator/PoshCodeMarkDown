---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 486
Published Date: 2008-07-28t18
Archived Date: 2016-12-06t06
---

# searching word - 

## Description

from sapien blog post by jeff hicks

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $word=New-Object -COM "Word.Application"
  
 $errorlog="c:\missing.csv"
 Set-Content $errorlog "Chapter,Script"
 Get-ChildItem c:\test\*.doc | foreach {
     $file=$_.fullname
     Write-Host $file
     $doc=$word.Documents.Open($file) 
     
     $style=$word.ActiveDocument.Styles | 
     where {$_.namelocal -eq "code Title"}
     
     $word.Selection.Find.Style = $style
     
     while ($word.Selection.Find.Execute()) {
         $text=($word.selection.sentences | select Text).text
         $script=(Join-Path "C:\scripts\posh" $text).Trim()
  
          if ((Get-Item $script -ea "silentlycontinue").exists) {
              Write-Host "verified $script"
              Move-Item $script -destination "c:\scripts\posh\ad"
          }
         else {
             $msg="{0},{1}" -f $file,$script
             Add-Content $errorlog $msg
         }
         
      } 
      
      $doc.close()
  
  }
  
 $word.quit()
`

