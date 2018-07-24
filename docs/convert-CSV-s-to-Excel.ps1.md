---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2123
Published Date: 2011-09-07t17
Archived Date: 2017-03-18t10
---

# convert csv/s to excel - 

## Description

this script allows you to convert one or more csvs into an excel file with each csv being a new worksheet in excel. the worksheet name will be the name of the file with the exception of the extension. so a file called test.csv will be named ‘test’. csvs can be piped into the function or you can use the -inputfile parameter to accomplish this.

## Comments



## Usage



## TODO



## function

`release-ref`

## Code

`#
 #
 Function Release-Ref ($ref) 
     {
         ([System.Runtime.InteropServices.Marshal]::ReleaseComObject(
         [System.__ComObject]$ref) -gt 0)
         [System.GC]::Collect()
         [System.GC]::WaitForPendingFinalizers() 
     }
 
 Function ConvertCSV-ToExcel
 {
   .SYNOPSIS  
     Converts one or more CSV files into an excel file.
      
   .DESCRIPTION  
     Converts one or more CSV files into an excel file. Each CSV file is imported into its own worksheet with the name of the
     file being the name of the worksheet.
        
   .PARAMETER inputfile
     Name of the CSV file being converted
   
   .PARAMETER output
     Name of the converted excel file
        
   .EXAMPLE  
   Get-ChildItem *.csv | ConvertCSV-ToExcel -output 'report.xlsx'
   
   .EXAMPLE  
   ConvertCSV-ToExcel -inputfile 'file.csv' -output 'report.xlsx'
     
   .EXAMPLE      
   ConvertCSV-ToExcel -inputfile @("test1.csv","test2.csv") -output 'report.xlsx'
   
   .NOTES
   Author: Boe Prox									      
   Date Created: 01SEPT210								      
   Last Modified:  
      
 #>
      
 [CmdletBinding(
     SupportsShouldProcess = $True,
     ConfirmImpact = 'low',
 	DefaultParameterSetName = 'file'
     )]
 Param (    
     [Parameter(
      ValueFromPipeline=$True,
      Position=0,
      Mandatory=$True,
      HelpMessage="Name of CSV/s to import")]
      [ValidateNotNullOrEmpty()]
     [array]$inputfile,
     [Parameter(
      ValueFromPipeline=$False,
      Position=1,
      Mandatory=$True,
      HelpMessage="Name of excel file output")]
      [ValidateNotNullOrEmpty()]
     [string]$output    
     )
 
 Begin {     
     [regex]$regex = "^\w\:\\"
     
     $count = ($inputfile.count -1)
    
     $excel = new-object -com excel.application
     
     $excel.DisplayAlerts = $False
 
     $excel.Visible = $False
 
     $workbook = $excel.workbooks.Add()
 
     $workbook.worksheets.Item(2).delete()
     $workbook.worksheets.Item(2).delete()   
 
     $i = 1
     }
 
 Process {
     ForEach ($input in $inputfile) {
         If ($i -gt 1) {
             $workbook.worksheets.Add() | Out-Null
             }
         $worksheet = $workbook.worksheets.Item(1)
         $worksheet.name = "$((GCI $input).basename)"
 
         If ($regex.ismatch($input)) {
             $tempcsv = $excel.Workbooks.Open($input) 
             }
         ElseIf ($regex.ismatch("$($input.fullname)")) {
             $tempcsv = $excel.Workbooks.Open("$($input.fullname)") 
             }    
         Else {    
             $tempcsv = $excel.Workbooks.Open("$($pwd)\$input")      
             }
         $tempsheet = $tempcsv.Worksheets.Item(1)
         $tempSheet.UsedRange.Copy() | Out-Null
         $worksheet.Paste()
 
         $tempcsv.close()
 
         $range = $worksheet.UsedRange
 
         $range.EntireColumn.Autofit() | out-null
         $i++
         } 
     }        
 
 End {
     $workbook.saveas("$pwd\$output")
 
     Write-Host -Fore Green "File saved to $pwd\$output"
 
     $excel.quit()  
 
     $a = Release-Ref($range)
     }
 }
`

