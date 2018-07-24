---
Author: krushna
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5682
Published Date: 2015-01-12t02
Archived Date: 2015-01-18t22
---

# ping -t using powershell - 

## Description

ping -t with list of servers names

## Comments



## Usage



## TODO



## function

`get-filename`

## Code

`#
 #
 <#
  http://blogs.technet.com/b/heyscriptingguy/archive/2009/09/01/hey-scripting-guy-september-1.aspx
 #>
 
 Function Get-FileName($initialDirectory)
 {  
  [System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") |
  Out-Null
 
  $OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
  $OpenFileDialog.initialDirectory = $initialDirectory
  $OpenFileDialog.filter = "All files (*.*)| *.*"
  $OpenFileDialog.ShowDialog() | Out-Null
  return($OpenFileDialog.filename)
 
 $inpoutfilepath =   Get-FileName -initialDirectory "c:\"
 if($inpoutfilepath -ne "")
 {
     if (Test-Path $inpoutfilepath)  #"D:\servers.txt"
     {
          $ComputerNames=Get-Content $inpoutfilepath
     
          foreach ($ComputerName in $ComputerNames) 
          {
             start-process cmd  " /c ping $ComputerName -t"
           }
     }
     else
     {
         Write-Warning "unable to find the input file"
     } 
 }
     else
     {
         Write-Warning "No File has been selected"
     }
`

