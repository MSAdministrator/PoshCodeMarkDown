---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3402
Published Date: 2012-05-07t14
Archived Date: 2017-05-22t03
---

# excel-loadfile - 

## Description

load an excel document into an excel com object for processing as input. these functions can be modified to start reading the data at a specified row count.  the data/headers do not need be on the first row.

## Comments



## Usage



## TODO



## function

`excel-loadfile`

## Code

`#
 #
 
 Function Excel-LoadFile {
 	Param (
 		$SourceFile,
 		$ExcelObject
 	)
 	
 	$Workbook   = $ExcelObject.Workbooks.Open($SourceFile)
 	$Worksheets = $Workbook.Worksheets
 	$Worksheet  = $Workbook.Worksheets.Item(1)
 	
 	return $Worksheet		
 }
 
 Function Excel-RowCount {
 	Param ($Worksheet)
 	
 	$range   = $Worksheet.UsedRange
 	$rows    = $range.Rows.Count
 	$rows    = $rows - 2
 	
 	return $rows
 }
 
 Function Excel-ColumnCount {
 	Param ($Worksheet)
 	
 	$range   = $Worksheet.UsedRange
 	$columns = $range.Columns.Count
 		
 	return $columns
 }
 
 Function Excel-ReadHeader {
     Param ($Worksheet)
 	
     $Headers =@{}
     $column = 1
     Do {
         $Header = $Worksheet.cells.item(3,$column).text
         If ($Header) {
             $Headers.add($Header, $column)
             $column++
         }
     } until (!$Header)
     $Headers
 }
 
 Function Excel-CloseFile {
 	Param($ExcelObject)
 	
 	$ExcelProcess = Get-Process Excel
 	$ExcelProcess | ForEach { Stop-Process ( $_.id ) }
 	[System.Runtime.Interopservices.Marshal]::ReleaseComObject($ExcelObject) | Out-Null
 }
`

