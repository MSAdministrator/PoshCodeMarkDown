---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6816
Published Date: 2017-03-23t02
Archived Date: 2017-03-25t17
---

# add-exceladdins.ps1 - add-exceladdins.ps1

## Description

this script will check if the specified microsoft office excel addins are loaded, and if not load them.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 $Addinfilename = 'Addin_01.xla'
 $Addinfilepath = 'C:\MyAddins\'
 $Excel = New-Object -ComObject excel.application
 $ExcelWorkbook = $excel.Workbooks.Add()
 if (($ExcelWorkbook.Application.AddIns | Where-Object {$_.name -eq $Addinfilename}) -eq $null) {
 $ExcelAddin = $ExcelWorkbook.Application.AddIns.Add("$Addinfilepath$Addinfilename", $True)
 $ExcelAddin.Installed = "True"
 Write-Host "$Addinfilename added"}
 else
 {Write-Host "$Addinfilename already added"}
 $Excel.Quit()
`

