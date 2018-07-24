---
Author: trevor
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3376
Published Date: 2012-04-19t19
Archived Date: 2012-12-29t04
---

# modify excel - 

## Description

powershell to modify excel

## Comments



## Usage



## TODO



## function

`release-ref`

## Code

`#
 #
 
 function Release-Ref ($ref) { 
 ([System.Runtime.InteropServices.Marshal]::ReleaseComObject( 
 [System.__ComObject]$ref) -gt 0) 
 [System.GC]::Collect() 
 [System.GC]::WaitForPendingFinalizers() 
 } 
  
 $objExcel = new-object -comobject excel.application  
 $objExcel.Visible = $True  
 $objWorkbook = $objExcel.Workbooks.Open("C:\Users\username\Desktop\scripts\groupmaps.xlsx") 
 
 #$objWorkbook.ActiveSheet.Cells.Item(2,2)= "Test a Write"
 
 #$content = $objWorkbook.ActiveSheet.Cells.Item(3,1).Text
 #"Cell B2 content: $content"
 $RowNum = 2
 
 While ($objWorkbook.ActiveSheet.Cells.Item($RowNum, 1).Text -ne "") {
  
     
     
     $lusername = $objWorkbook.ActiveSheet.Cells.Item($RowNum,2).Text
     Import-Csv C:\Users\username\Desktop\scripts\usermaps.csv | foreach {
 		if ($lusername -eq $_.ousername){
 			$objWorkbook.ActiveSheet.Cells.Item($RowNum,4)= $_.nusername
 			$objWorkbook.ActiveSheet.Cells.Item($RowNum,5)= $_.DisplayName
 		}
 	}
 	           
     $RowNum++
 }
  
  
 $a = Release-Ref($objWorkbook) 
 $a = Release-Ref($objExcel)
`

