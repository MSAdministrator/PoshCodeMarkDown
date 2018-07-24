---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3135
Published Date: 2012-12-30t16
Archived Date: 2012-01-11t06
---

# update-isetabs - 

## Description

reloads all the file tabs in ise (ps3ctp2)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 foreach($tab in $psISE.PowerShellTabs) {
    foreach($file in $tab.Files) {
       $position = Select-Object -InputObject $file.Editor -Property CaretLine, CaretColumn
       $content = Get-Content $file.FullPath -Raw
       if($content -ne $file.Editor.Text) {
          $file.Editor.Text = Get-Content $file.FullPath -Raw
          Write-Verbose "Updated $($file.DisplayName)"
          $file.Editor.SetCaretPosition( $Position.CaretLine, $Position.CaretColumn )
       }
    }
 }
`

