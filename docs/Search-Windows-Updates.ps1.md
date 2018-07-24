---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4482
Published Date: 2013-09-22t10
Archived Date: 2013-09-29t23
---

# search windows updates - search-windowsupdateslocal.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`search-windowsupdateslocal`

## Code

`#
 #
 #############################################################################################
 #
 #
 #
 
 Function Search-WindowsUpdatesLocal ([String] $Search)
 {
 $Search = $Search + "\d*" 
 $Searcher = New-Object -comobject Microsoft.Update.Searcher
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
     $SearchAnswer = $string | Select-String -Pattern $Search | Select-Object { $_.Matches } 
      $output = New-Object -TypeName PSobject
      $output | add-member NoteProperty �Date� -value $Update.Date
      $output | add-member NoteProperty �HotFixID� -value $SearchAnswer.� $_.Matches �.Value
      $output | Add-Member NoteProperty "Result" -Value $Result
      if($output.HotFixID)
      {
      $OutputCollection += $output
      }
  
     }
 
     $OutputCollection
     }
`

