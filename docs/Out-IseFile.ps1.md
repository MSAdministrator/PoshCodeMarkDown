---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1623
Published Date: 
Archived Date: 2010-02-26t16
---

# out-isefile - 

## Description

powershells wrapping behaviour may be adequate for the console host. using ise it’s far from the optimal. but you don’t need to use the output pane. send your output to a new ise editor. to see the difference just try

## Comments



## Usage



## TODO



## function

`out-isefile`

## Code

`#
 #
 
 
 function Out-IseFile
 {
 
     [CmdletBinding()]
     param (
         [Parameter(Position = 0, Mandatory = $True,  ValueFromPipeline = $True) ]        
         $msg, 
         [Parameter(Position = 1, Mandatory = $False, ValueFromPipeline = $False)]        
         $path = $null,
         [Parameter(Position = 2, Mandatory = $False, ValueFromPipeline = $False)]        
         $width = 2000,
         [switch]$fl,
         [switch]$passEditor
     )
 
     Begin
     {
         $cnt = 0
         $count   = $psise.CurrentPowerShellTab.Files.count
         $null    = $psIse.CurrentPowerShellTab.Files.Add()
         $Newfile = $psIse.CurrentPowerShellTab.Files[$count]
         if ($path)
         {    
             $NewFile.SaveAs($path)
             $NewFile.Save([Text.Encoding]::default)
         }
         $Editor = $Newfile.Editor
         
         filter Remove-TrailingBlanks { $_ -replace '(?m)\s*$', '' }
     }
 
     Process
     {
         $cnt++
        if ($cnt -eq 1)
        {
         $cmsg = @($msg)
        }
        else
        {
        $cmsg = $cmsg +  @($msg)
        }
     }
     
     End
     {   
         if ($cnt -gt 1)
         {
             $msg = $cmsg
         }
         if ($fl)
         {
             $msg = $msg |  Format-list | out-string -width $width | Remove-trailingBlanks
         }
         else
         {
             $msg = $msg | out-string -width $width | Remove-trailingBlanks
         }
         $msg = $msg +  "`r`n"
 
         #$Editor.SetCaretPosition($Editor.LineCount, 1)
         $Editor.InsertText($msg)
 
         if ($passEditor){ $Newfile }
         
         Write-Verbose "Anzahl = $cnt   len msg = $($msg.count)"
     }
 }
 
 <#
 
 $profile.psextended | out-IseFile -fl -verb
 
 get-help *  | out-string -width 2000| out-ISEFile -verb
 get-help *  | out-ISEFile -verb
 get-help *  | out-ISEFile 
 
  
  
 $a =  $profile.psextended
 out-IseFile  $a -fl
 
 
 $a = get-help *  | out-string -width 2000 
 out-IseFile  $a
 #>
`

