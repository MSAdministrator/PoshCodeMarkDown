---
Author: travis kuntz <tkuntz2 at google's mail service>
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5700
Published Date: 2016-01-21t18
Archived Date: 2016-10-05t05
---

# convert csvs to xlsxs - 

## Description

these functions convert a csv file into an xlsx file.

## Comments



## Usage



## TODO



## function

`get-filename`

## Code

`#
 #
 ################################################
 #
 #
 #
 
 function Get-FileName($searchRoot){
   .SYNOPSIS
   This function provides GUI access to select a file path and name for a Powershell script.
   
   .DESCRIPTION
   This function is called through other functions.  It displays a popup and allows a user to choose a file.
 
   That file name and path can then be passed as a variable for insertion into another function, i.e. $var = get-filename.
   .EXAMPLE
   $somevariable = get-filename
   .EXAMPLE
   $var = get-filename | get-acl -path $var
   .PARAMETER initialDirectory
   This optional parameter can specify the directory in which the file browsing window starts.
 #>
  
  [System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") |
  
     Out-Null
 
          $OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
          $OpenFileDialog.initialDirectory = $searchRoot
          $OpenFileDialog.filter = "All files (*.*)| *.*"
          $OpenFileDialog.ShowDialog() | Out-Null
          $OpenFileDialog.filename
 
 
 function Convert-CSV {
  <#
   .SYNOPSIS
     This function converts a *.csv file to a *.xlsx file.
   .DESCRIPTION
   Excel files, in order to adhere to an open-source standard, changed from a COM oriented file type to an XML file type.
 
   For that reason, this script initiates a COM object and silently launches Excel.
 
   Once Excel brings in the worksheet, we initiate a save through the API and silently close Excel.
   .EXAMPLE
   Type convert-csv and press enter.  A browse dialogue box is opened.  Choose your *.csv file.  The default save location is the original folder.
   .EXAMPLE
   convert-csv -filename c:\results\test.csv -outpath c:\scripts
   .EXAMPLE
   convert-csv c:\results\test.csv c:\scripts
   .PARAMETER filename
   The source CSV file you would like to convert to XLSX.
   .PARAMETER outpath
   The path in which you want the new XLSX file to be placed.
     #>
     [CmdletBinding()]
       param
       (
         [Parameter(Position=0)]
         [string]$filename,	
         [string]$outpath
       )
      
 if ([string]::IsNullOrEmpty($filename)) {
        $filename = get-filename -searchroot "c:\"
     }
         
 
     $xl = new-object -comobject excel.application
 
     $xl.visible = $false
 
         $Workbook = $xl.workbooks.open($filename)
 
         $Worksheets = $Workbooks.worksheets
 
     $xl.columns.item(1).columnWidth = 27
     $xl.columns.item(2).columnWidth = 27
     $xl.columns.item(3).columnWidth = 27
     $xl.columns.item(4).columnWidth = 27
 
     if ([string]::IsNullOrEmpty($outpath)) {
        $outpath = (split-path $filename -parent)
     }
 
     $filepath = split-path $filename -parent
     $basename = (get-item -path $filename).BaseName
     $excelfile = ("$outpath" +"\" + "$basename" + ".xlsx")
     
 
         $Workbook.SaveAs("$excelfile",51)
         $Workbook.Saved = $True
 
     $xl.Quit()
     
`

