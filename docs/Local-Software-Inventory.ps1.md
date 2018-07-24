---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2940
Published Date: 2011-08-31t14
Archived Date: 2015-11-26t04
---

# local software inventory - 

## Description

this script will create (using com) an ms excel spreadsheet of all the software installed on the local machine. basic information about the software, as provided in the registry, is also included. when the script has completed the data collection, all columns are set to “auto fit” for width and blank rows in the spreadsheet are removed.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Excel = New-Object -ComObject Excel.Application
 $Excel.Visible = $True
 $Excel.DisplayAlerts = $False
 
 $intRow = 1
 
 $Excel = $Excel.Workbooks.Add()
 $Sheet = $Excel.Worksheets.Item(1)
 
 $Sheet.Cells.Item($intRow,1)  = "Software Vendor"
 $Sheet.Cells.Item($intRow,1).Font.Bold = $True
 $Sheet.Cells.Item($intRow,2)  = "Software Name"
 $Sheet.Cells.Item($intRow,2).Font.Bold = $True
 $Sheet.Cells.Item($intRow,3)  = "Version"
 $Sheet.Cells.Item($intRow,3).Font.Bold = $True
 $Sheet.Cells.Item($intRow,4)  = "Estimated Size (KB)"
 $Sheet.Cells.Item($intRow,4).Font.Bold = $True
 $Sheet.Cells.Item($intRow,5)  = "Installation Date"
 $Sheet.Cells.Item($intRow,5).Font.Bold = $True
 
 $Results = Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*
 
 foreach ($objResult in $Results)
 {
 	$intRow++
 	$Name = $objResult.DisplayName
 	$SizeKB = $objResult.EstimatedSize
 	$Version = $objResult.DisplayVersion
 	$InstDate = $objResult.InstallDate
 	$Vendor = $objResult.Publisher
 	$Sheet.Cells.Item($intRow,1) = $Vendor
 	$Sheet.Cells.Item($intRow,2) = $Name
 	$Sheet.Cells.Item($intRow,3) = $Version
 	$Sheet.Cells.Item($intRow,4) = $SizeKB
 	$Sheet.Cells.Item($intRow,5) = $InstDate
 }
 
 $Sheet.UsedRange.EntireColumn.AutoFit()
 
 $i = 1
 
 $xlCellTypeLastCell = 11
 $used = $Sheet.usedRange
 $lastCell = $used.SpecialCells($xlCellTypeLastCell)
 $row = $lastCell.row
 for ($i = 1; $i -le $row; $i++)
 {
 	If ($Sheet.Cells.Item($i, 1).Value() -eq $Null)
 	{
 		$Range = $Sheet.Cells.Item($i, 1).EntireRow
 		$Range.Delete()
 	}
 }
`

