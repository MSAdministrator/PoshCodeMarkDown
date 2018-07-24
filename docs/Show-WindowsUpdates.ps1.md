---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4484
Published Date: 2016-09-23t22
Archived Date: 2016-05-30t19
---

# show-windowsupdates - show-windowsupdateslocal.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-windowsupdateslocal`

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 
 Function Show-WindowsUpdatesLocal
 {
     $Searcher = New-Object -ComObject Microsoft.Update.Searcher
     $History = $Searcher.GetTotalHistoryCount()
     $Updates =  $Searcher.QueryHistory(1,$History)
 
     $OutputCollection=  @()
     
     Foreach ($update in $Updates)
         {
         $Result = $null
         Switch ($update.ResultCode)
             {
             0 { $Result = 'NotStarted'}
             1 { $Result = 'InProgress' }
             2 { $Result = 'Succeeded' }
             3 { $Result = 'SucceededWithErrors' }
             4 { $Result = 'Failed' }
             5 { $Result = 'Aborted' }
             default { $Result = $_ }
             }
     $string = $update.title
     $Regex = �KB\d*�
     $KB = $string | Select-String -Pattern $regex | Select-Object { $_.Matches }
     $output = New-Object -TypeName PSobject
     $output | add-member NoteProperty �Date� -value $Update.Date
     $output | add-member NoteProperty �HotFixID� -value $KB.� $_.Matches �.Value
     $output | Add-Member NoteProperty "Result" -Value $Result
     $output | add-member NoteProperty �Title� -value $string
     $output | add-member NoteProperty �Description� -value $update.Description
 
      $OutputCollection += $output
  
     }
 
     $OutputCollection
     }
`

