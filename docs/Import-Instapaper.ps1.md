---
Author: adminian
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2588
Published Date: 2011-03-27t18
Archived Date: 2016-11-14t08
---

# import-instapaper - 

## Description

search and browse to url in an instapaper csv.

## Comments



## Usage



## TODO



## script

`get-filename`

## Code

`#
 #
 
 
 
 Function Get-FileName($initialDirectory)
 {   
  [System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") |
  Out-Null
 
  $OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
  $OpenFileDialog.ShowHelp = $true
  $OpenFileDialog.initialDirectory = $initialDirectory
  $OpenFileDialog.filter = "All files (*.*)| *.*"
  $OpenFileDialog.ShowDialog() | Out-Null
  $OpenFileDialog.filename
 }
 
 $fileName = Get-FileName -initialDirectory "~"
 
 $csv = Import-Csv $fileName
 
 $search = Read-Host "Search Topic: "
 $csv | Where-Object {$_.Title -like "*" + $search + "*"} | ft title
 
 $search = Read-Host "Open Title: "
 $result = $csv | Where-Object {$_.Title -like "*" + $search + "*"}
 $ie = New-Object -comobject "InternetExplorer.Application"
 $ie.visible = $true
 $ie.navigate($result.url)
`

