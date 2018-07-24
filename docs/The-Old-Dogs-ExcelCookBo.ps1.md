---
Author: kent finkle
Publisher: 
Copyright: 
Email: 
Version: 2009.01
Encoding: utf-8
License: cc0
PoshCode ID: 1974
Published Date: 2010-07-15t13
Archived Date: 2016-09-07t04
---

# the old dogs excelcookbo - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments

this is an updated version of my cook book from augest of 2008

## Usage



## TODO



## script

`using-culture`

## Code

`#
 #
 #
 #
 #===============================================================
 #
 #
 #
 #
 
 Function Using-Culture (
 [System.Globalization.CultureInfo]$culture,
 [ScriptBlock]$script)
 {
     $OldCulture = [System.Threading.Thread]::CurrentThread.CurrentCulture
     trap
     {
     [System.Threading.Thread]::CurrentThread.CurrentCulture = $OldCulture
     }
     [System.Threading.Thread]::CurrentThread.CurrentCulture = $culture
     $ExecutionContext.InvokeCommand.InvokeScript($script)
     [System.Threading.Thread]::CurrentThread.CurrentCulture = $OldCulture
 
 Using-Culture en-us {
 $xl= New-Object -COM Excel.Application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $xl.Workbooks.Open("C:\Scripts\PowerShell\test.xls")
 #$xl.Quit()
 #[System.Runtime.InteropServices.Marshal]::ReleaseComObject($xl)
 }
  
 $xl = new-object -comobject excel.application 
 
 
 $xl.Visible = $true
 
 #
  
 $wb = $xl.Workbooks.Add()
  
 #
 #
 #
 $xl = New-Object -comobject Excel.Application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.Workbooks.Open("C:\Scripts\powershell\test.xls")
 
 #
  
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.Workbooks.Open("C:\Scripts\test.xls")
 $ws = $xl.Sheets.Add()
 
 #
 #
 $xl = New-Object -comobject Excel.Application
 $xl.visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.workbooks.open("C:\Scripts\PowerShell\test.xls")
 $ws1 = $wb.worksheets | where {$_.name -eq "sheet1"} #<------- Selects sheet 1
 $ws2 = $wb.worksheets | where {$_.name -eq "sheet2"} #<------- Selects sheet 2
 $ws3 = $wb.worksheets | where {$_.name -eq "sheet3"} #<------- Selects sheet 3
 $ws1.activate()
 Start-Sleep 1 
 $ws2.activate()
 Start-Sleep 1 
 $ws3.activate()
  
 $ws1.cells.Item(1,1).Value() = "x" 
 
 #
 #
 
 $xlCellTypeLastCell = 11
 
 $used = $ws.usedRange 
 $lastCell = $used.SpecialCells($xlCellTypeLastCell) 
 $row = $lastCell.row 
 
 for ($i = 1; $i -le $row; $i++) {
 If ($ws1.cells.Item(1,2).Value() = "y") {
           }
 
 }
 
 #
 $mainRng = $ws.usedRange
 $mainRng.Select()
 $objSearch = $mainRng.Find("Grand Total")
 $objSearch.Select()
 
 #
 #
 
 Function DeleteEmptyRows {
 $used = $ws.usedRange 
 $lastCell = $used.SpecialCells($xlCellTypeLastCell) 
 $row = $lastCell.row 
 
 for ($i = 1; $i -le $row; $i++) {
     If ($ws.Cells.Item($i, 1).Value() -eq $Null) {
         $Range = $ws.Cells.Item($i, 1).EntireRow
         $Range.Delete()
         }
     }
 } 
 
 $xlCellTypeLastCell = 11 
 
 $xl = New-Object -comobject excel.application 
 $xl.Visible = $true 
 $xl.DisplayAlerts = $False 
 
 
  
 
 #
 #
  
 $ws1.range("a${Sumrow}:b$Sumrow").font.bold = "true" 
 
 #
  
 $range4=$ws.range("3:3")
 $range4.cells="Row 3"
 
 #
  
 $wb.Name
 
 #
 
 $xlCellTypeLastCell = 11
 $used = $ws.usedRange
 $lastCell = $used.SpecialCells($xlCellTypeLastCell)
 $lastrow = $lastCell.row 
 
 
 $mainRng = $ws1.UsedRange.Cells 
 $RowCount = $mainRng.Rows.Count  
 $R = $RowCount 
 
 #
  
 $mainRng = $ws1.UsedRange.Cells 
 $ColCount = $mainRng.Columns.Count 
 $RowCount = $mainRng.Rows.Count  
 $xRow = $RowCount
 $xCol = $ColCount 
 
 #
  
 $xl = New-Object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 for ($row = 1; $row -lt 11; $row++)
 {
     $ws.Cells.Item($row,1) = $row
 }
 #
  
 $m = (get-date).month
 $d = (get-date).day
 $y = [string] (get-date).year
 $y = $y.substring($y.length - 2, 2)
 $f = "C:\Scripts\" + $m + "-" + $d + "-" + $y + ".xlsx"
 
 #
 #
  
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $row = 1
 $s = dir Z:\MBSA_Report\ScanData\*.mbsa
 $s | foreach -process {
     $ws.Cells.Item($row,1) = $_;
     $row++
 }
 
 #
  
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $row = 1
 dir Z:\MBSA_Report\ScanData\*.mbsa |
 ForEach-Object {
 $FileName = $_.name
 $N = $FileName.tostring()
 $E = $N.split()
 $F = $E[2]
 $ws.Cells.Item($row,1) = $F;
     $row++
 }
 $ws.Cells.Item($row,1) = $_;
 
 #
  
 function Release-Ref ($ref) {
 #[System.Runtime.Interopservices.Marshal]::ReleaseComObject($xl)
 [System.Runtime.InteropServices.Marshal]::ReleaseComObject($ref)
 [System.GC]::Collect()
 [System.GC]::WaitForPendingFinalizers()
 }
 $xl = New-Object -comobject Excel.Application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $range = $ws.Cells.Item(1,1)
 $row = 1
 $s = Get-Process | Select-Object name
 $s | foreach -process {
     $range = $ws.Cells.Item($row,1);
     $range.Value = $_.Name;
     $row++ } 
 $wb.SaveAs("C:\Scripts\Get_Process.xls")
 Release-Ref $range
 Release-Ref $ws
 Release-Ref $wb
 $xl.Quit()
 Release-Ref $xl
 #***For a remote machine try
 $strComputer = (remote machine name)
 $P = gwmi win32_process -comp $strComputer
 
 #
  
 function Release-Ref ($ref) {
 #[System.Runtime.Interopservices.Marshal]::ReleaseComObject($xl)
 [System.Runtime.InteropServices.Marshal]::ReleaseComObject($ref)
 [System.GC]::Collect()
 [System.GC]::WaitForPendingFinalizers()
 }
 
 $xl = New-Object -comobject Excel.Application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $excel.Workbooks.Add()
 $ws = $workbook.Worksheets.Item(1)
 $range = $worksheet.Cells.Item(1,1)
 $row = 1
 $s = Get-History | Select-Object CommandLine $s | foreach -process { `
 $range = $worksheet.Cells.Item($row,1); `
 $range.Value = $_.CommandLine; `
 $row++ }
 $xl.DisplayAlerts = $False
 $wb.SaveAs("C:\Scripts\Get_CommandLine.xls")
 Release-Ref $range
 Release-Ref $ws
 Release-Ref $wb
 $xl.Quit()
 Release-Ref $xl
 
 #
  
 $s = gc C:\Scripts\Test.txt
 $s = $s -replace("~","`t")
 $s | sc C:\Scripts\Test.txt
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.Workbooks.Open("C:\Scripts\Test.txt")
 
 #
 #
 #-----------------------------------------------------
 
 function Release-Ref ($ref) {
 ([System.Runtime.InteropServices.Marshal]::ReleaseComObject(
 [System.__ComObject]$ref) -gt 0)
 [System.GC]::Collect()
 [System.GC]::WaitForPendingFinalizers()
 $xlValidateWholeNumber = 1
 $xlValidAlertStop = 1
 $xlBetween = 1
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $r = $ws.Range("e5")
 $r.Validation.Add($xlValidateWholeNumber,$xlValidAlertStop, $xlBetween, "5", "10")
 $r.Validation.InputTitle = "Integers"
 $r.Validation.ErrorTitle = "Integers"
 $r.Validation.InputMessage = "Enter an integer from five to ten"
 $r.Validation.ErrorMessage = "You must enter a number from five to ten" 
 $a = Release-Ref $r
 $a = Release-Ref $ws
 $a = Release-Ref $wb
 $a = Release-Ref $xl
 
 #
  
 $xrow = 1
 $yrow = 8
 $xl = New-Object -c excel.application
 $xl.visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.workbooks.add()
 $ws = $wb.sheets.item(1)
 1..8 | % { $ws.Cells.Item(1,$_) = $_ }
 1..8 | % { $ws.Cells.Item(2,$_) = 9-$_ }
 $range = $ws.range("a${xrow}:h$yrow")
 $range.activate
 $ch.chartType = 58
 $ch.setSourceData($range)
 $ch.export("C:\test.jpg")
 $xl.quit() 
 1..48 | % {$ch.chartStyle = $_; $xl.speech.speak("Style $_"); sleep 1}
 
 
 Function XLcharts {
 $xlColumnClustered = 51
 $xlColumns = 2
 $xlLocationAsObject = 2
 $xlCategory = 1
 $xlPrimary = 1
 $xlValue = 2
 $xlRows = 1
 $xlLocationAsNewSheet = 1
 $xlRight = -4152
 $xlBuiltIn =21
 $xlCategory = 1 
 $xl = New-Object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $ws = $wb.Sheets.Add()
 $ws.Cells.Item(1, 2) =  "Jan"
 $ws.Cells.Item(1, 3) =  "Feb"
 $ws.Cells.Item(1, 4) =  "Mar"
 $ws.Cells.Item(2, 1) =  "John"
 $ws.Cells.Item(3, 1) =  "Mae"
 $ws.Cells.Item(4, 1) =  "Al"
 $ws.Cells.Item(2, 2) =  100
 $ws.Cells.Item(2, 3) =  200
 $ws.Cells.Item(2, 4) =  300
 $ws.Cells.Item(3, 2) =  400
 $ws.Cells.Item(3, 3) =  500
 $ws.Cells.Item(3, 4) =  600
 $ws.Cells.Item(4, 2) =  900
 $ws.Cells.Item(4, 3) =  800
 $ws.Cells.Item(4, 4) =  700 
 $Range = $ws.range("A1:D4")
 $ch = $xl.charts.add()
 $ch.chartType = 58
 $ch.name ="Bar Chart"
 $ch.Tab.ColorIndex = 3
 $ch.setSourceData($Range)
 [void]$ch.Location, $xlLocationAsObject, "Bar Chart"
 $ch.HasTitle = $False
 $ch.Axes($xlCategory, $xlPrimary).HasTitle = $False
 $ch.Axes($xlValue, $xlPrimary).HasTitle = $False 
 $ch2 = $xl.Charts.Add() | Out-Null
 $ch2.HasTitle = $true
 $ch2.ChartTitle.Text = "Sales"
 $ch2.Axes($xlCategory).HasTitle = $true
 $ch2.Axes($xlCategory).AxisTitle.Text = "1st Quarter"
 $ch2.Axes($xlValue).HasTitle = $True
 $ch2.Axes($xlValue).AxisTitle.Text = "Dollars"
 [void]$ch2.Axes($xlValue).Select
 $ch2.name ="Columns with Depth"
 $ch2.Tab.ColorIndex = 5
 [void]$ws.cells.entireColumn.Autofit()
 #
 
 $xl.visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.workbooks.add() 
 $ws = $wb.Worksheets.Item(1)
 #
  
 $xlSummaryAbove = 0
 $xlSortValues = $xlPinYin = 1
 $xlAscending = 1
 $xlDescending = 2
 $xlYes = 1 
 $xl = New-Object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1) 
 $r = $ws.UsedRange
 $a = $r.sort($r2, $xlAscending) 
 $r = $ws.UsedRange
 $r2 = $ws.Range("B2")
 $r3 = $ws.Range("E2")
 $a = $r.sort($r2, $xlAscending, $r3,$null, $xlAscending) 
 $r = $ws1.UsedRange
 $r2 = $ws1.Range("B2")
 $r3 = $ws1.Range("E2")
 $r4 = $ws1.Range("C2") 
 $a = $r.Sort($r2,$xlDescending,$r3,$null,$xlAscending, `
 $r4,$xlAscending,$xlYes)
 
 #-----------------------------------------------------
  
 $xlAutomatic=-4105
 $xlBottom = -4107
 $xlCenter = -4108
 $xlContext = -5002
 $xlContinuous=1
 $xlDiagonalDown=5
 $xlDiagonalUp=6
 $xlEdgeBottom=9
 $xlEdgeLeft=7
 $xlEdgeRight=10
 $xlEdgeTop=8
 $xlInsideHorizontal=12
 $xlInsideVertical=11
 $xlNone=-4142
 $xlThin=2 
 $xl = new-object -com excel.application
 $xl.visible=$true
 $wb = $xl.workbooks.open("d:\book1.xls")
 $ws = $wb.worksheets | where {$_.name -eq "sheet1"}
 $selection = $ws.range("A1:D1")
 $selection.select() 
 $selection.HorizontalAlignment = $xlCenter
 $selection.VerticalAlignment = $xlBottom
 $selection.WrapText = $false
 $selection.Orientation = 0
 $selection.AddIndent = $false
 $selection.IndentLevel = 0
 $selection.ShrinkToFit = $false
 $selection.ReadingOrder = $xlContext
 $selection.MergeCells = $false
 $selection.Borders.Item($xlInsideHorizontal).Weight = $xlThin 
  
  
 [reflection.assembly]::loadWithPartialname("Microsoft.Office.Interop.Excel") |
 Out-Null
 $xlConstants = "microsoft.office.interop.excel.Constants" -as [type] 
  
 $ws.columns.item("F").HorizontalAlignment = $xlConstants::xlCenter
 $ws.columns.item("K").HorizontalAlignment = $xlConstants::xlCenter
 
  
  
 $lineStyle = "microsoft.office.interop.excel.xlLineStyle" -as [type]
 $colorIndex = "microsoft.office.interop.excel.xlColorIndex" -as [type]
 $borderWeight = "microsoft.office.interop.excel.xlBorderWeight" -as [type]
 $chartType = "microsoft.office.interop.excel.xlChartType" -as [type] 
 
 For($b = 1 ; $b -le 2 ; $b++)
 {
 $ws.cells.item(1,$b).font.bold = $true
 $ws.cells.item(1,$b).borders.LineStyle = $lineStyle::xlDashDot
 $ws.cells.item(1,$b).borders.ColorIndex = $colorIndex::xlColorIndexAutomatic
 $ws.cells.item(1,$b).borders.weight = $borderWeight::xlMedium
 }
 $workbook.ActiveChart.chartType = $chartType::xl3DPieExploded
 $workbook.ActiveChart.SetSourceData($range) 
 
 #
  
 $xlFillWeekdays = 6 
 $xl = New-Object -com excel.application
 $xl.visible=$true
 $wb = $xl.workbooks.add()
 $ws = $wb.worksheets | where {$_.name -eq "sheet1"}
 $range1= $ws.range("A1")
 $range1.value() = (get-date).toString("d")
 $range2 = $ws.range("A1:A25")
 $range1.AutoFill($range2,$xlFillWeekdays)
 $range1.entireColumn.Autofit() 
 $xlCellTypeLastCell = 11
 $xl = new-object -com excel.application
 $xl.visible=$true
 $wb = $xl.workbooks.add()
 $ws = $wb.worksheets | where {$_.name -eq "sheet1"} 
 $used = $ws.usedRange
 $lastCell = $used.SpecialCells($xlCellTypeLastCell)
 $lastrow = $lastCell.row 
 $ws.Cells.Item(2,1).FormulaR1C1 = "=CONCATENATE(C[+1],C[+2],C[+3])"
 $range1= $ws.range("A2")
 $r = $ws.Range("A2:A$lastrow")
 $range1.AutoFill($r) | Out-Null
 [void]$range1.entireColumn.Autofit() 
 $wb.close()
 $xl.quit()
 
 #
  
 $xl = new-object -comobject Excel.Application 
 $xl.visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.workbooks.open("C:\Scripts\powershell\test.xls") 
  
 $ws = $wb.worksheets | where {$_.name -eq "sheet1"} 
  
 $range = $ws.range("A1:B1")
 $range.font.bold = "true" 
  
 $range2 = $ws.range("A2:B2")
 $range2.font.italic = "true" 
  
 $range4=$ws.range("3:3")
 $range4.cells="Row 3"
 
 $range3=$ws.range("A2:B2")
 $Range3.font.size=24 
 
 
 $range4=$ws.range("3:3")
 
 $range4.font.italic="$true"
 $range4.font.bold=$true
 $range4.font.size=10
 $range4.font.name="Comic Sans MS" 
 
 #
  
 $xl = New-Object -com Excel.Application
 $xl.visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $ws.Cells.Item(1,1) = â��A value in cell A1.â��
 [void]$ws.Range("A1").AddComment()
 [void]$ws.Range("A1").comment.Visible = $False
 [void]$ws.Range("A1").Comment.text("OldDog: `r this is a comment")
 [void]$ws.Range("A2").Select 
 
 #
 
 $xl = New-Object -comobject Excel.Application
 $xl.visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1) 
 $used = $ws.usedRange
 $lastCell = $used.SpecialCells($xlCellTypeLastCell)
 $row = $lastCell.row
 $range = $ws.UsedRange
 [void]$ws.Range("A8:F$row").Copy()
 [void]$ws.Range("A8").PasteSpecial(-4163) 
 
 #
 #****************************************** 
  
 $m = (get-date).month
 $d = (get-date).day
 $y = [string] (get-date).year
 $y = $y.substring($y.length - 2, 2)
 $f = "C:\Scripts\" + $m + "-" + $d + "-" + $y + ".xlsx" 
 $xl = New-Object -comobject Excel.Application
 $xl.visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1) 
 for ($i = 0; $i -le 8; $i++) {
 $ws = $wb.Sheets.Add() } 
 $xl.Worksheets.item(1).name = "Jan"
 $xl.Worksheets.item(2).name = "Feb"
 $xl.Worksheets.item(3).name = "Mar"
 $xl.Worksheets.item(4).name = "Apr"
 $xl.Worksheets.item(5).name = "May"
 $xl.Worksheets.item(6).name = "June"
 $xl.Worksheets.item(7).name = "July"
 $xl.Worksheets.item(8).name = "Aug"
 $xl.Worksheets.item(9).name = "Sept"
 $xl.Worksheets.item(10).name = "Oct"
 $xl.Worksheets.item(11).name = "Nov"
 $xl.Worksheets.item(12).name = "Dec" 
 $wb.SaveAs($F) 
 
 #*******************************************
  
 Function XLFindDups {
 $xlExpression = 2
 $xlPasteFormats = -4122
 $xlNone = -4142
 $xlToRight = -4161
 $xlToLeft = -4159
 $xlDown = -4121
 $xlShiftToRight = -4161
 $xlFillDefault = 0
 $xlSummaryAbove = 0
 $xlSortValues = $xlPinYin = 1
 $xlAscending = 1
 $xlDescending = 2
 $xlYes = 1
 $xlTopToBottom = 1
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $ws.name = "Concatenate" 
 $ws.Tab.ColorIndex = 4 
 $ws.Cells.Item(1,1) = "FirstName"
 $ws.Cells.Item(1,2) = "MI"
 $ws.Cells.Item(1,3) = "LastName"
 $ws.Cells.Item(2,1) = "Jesse"
 $ws.Cells.Item(2,2) = "L"
 $ws.Cells.Item(2,3) = "Roberts"
 $ws.Cells.Item(3,1) = "Mary"
 $ws.Cells.Item(3,2) = "S"
 $ws.Cells.Item(3,3) = "Talbert"
 $ws.Cells.Item(4,1) = "Ben"
 $ws.Cells.Item(4,2) = "N"
 $ws.Cells.Item(4,3) = "Smith"
 $ws.Cells.Item(5,1) = "Ed"
 $ws.Cells.Item(5,2) = "S"
 $ws.Cells.Item(5,3) = "Turner"
 $ws.Cells.Item(6,1) = "Mary"
 $ws.Cells.Item(6,2) = "S"
 $ws.Cells.Item(6,3) = "Talbert"
 $ws.Cells.Item(7,1) = "Jesse"
 $ws.Cells.Item(7,2) = "L"
 $ws.Cells.Item(7,3) = "Roberts"
 $ws.Cells.Item(8,1) = "Joe"
 $ws.Cells.Item(8,2) = "L"
 $ws.Cells.Item(8,3) = "Smith"
 $ws.Cells.Item(9,1) = "Ben"
 $ws.Cells.Item(9,2) = "A"
 $ws.Cells.Item(9,3) = "Smith" 
 $used = $ws.usedRange
 $lastCell = $used.SpecialCells($xlCellTypeLastCell)
 $lastrow = $lastCell.row 
 $range4=$ws.range("2:2")
 $range4.Select() | Out-Null
 $xl.ActiveWindow.FreezePanes = $true
 $ws.cells.EntireColumn.AutoFit() | Out-Null
 $range1 = $ws.Range("A1").EntireColumn
 $range1.Insert($xlShiftToRight) | Out-Null
 $range1.Select() | Out-Null
 $ws.Cells.Item(1, 1) = "Concat"
 $r2 = $ws.Range("A2")
 $r2.Select() | Out-Null
 $ws.Cells.Item(2,1).FormulaR1C1 = "=CONCATENATE(C[+1],C[+2],C[+3])"
 $range1= $ws.range("A2")
 $r = $ws.Range("A2:A$lastrow")
 $range1.AutoFill($r) | Out-Null
 $range.EntireColumn.AutoFit() | Out-Null
 $select = $range1.SpecialCells(11).Select()  | Out-Null
 $ws.Range("A2:A$lastrow").Copy()| Out-Null
 $ws1 = $wb.Sheets.Add()
 $ws1.name = "FindDups"
 $ws1 = $wb.worksheets | where {$_.name -eq "FindDups"}
 $ws1.Tab.ColorIndex = 5
 $ws1.Select() | Out-Null
 [void]$ws1.Range("A2").PasteSpecial(-4163)
 $ws1.Range("A1").Select() | Out-Null 
 $objRange = $xl.Range("B1").EntireColumn
 [void] $objRange.Insert($xlShiftToRight) 
 $ws1.Cells.Item(1, 2) = "Dups"
 $range = $ws.range("B1:D$lastrow")
 $range.copy() | Out-Null
 [void]$ws1.Range("C1").PasteSpecial(-4163) 
 $ws1.Cells.Item(2,2).FormulaR1C1 = "=COUNTIF(C[-1],RC[-1])>1"
 $range1= $ws1.range("B2")
 $range2 = $ws1.range("B2:B$lastrow")
 [void]$range1.AutoFill($range2,$xlFillDefault) 
 $xl.Range("B2").Select() | Out-Null
 $xl.Selection.FormatConditions.Delete()
 $xl.Selection.FormatConditions.Add(2, 0, "=COUNTIF(A:A,A2)>1") | Out-Null
 $xl.Selection.FormatConditions.Item(1).Interior.ColorIndex = 8
 $xl.Selection.Copy() | Out-Null
 $xl.Columns.Item("B:B").Select() | Out-Null
 $xl.Range("B2").Activate() | Out-Null
 $xl.Selection.PasteSpecial(-4122, -4142, $false, $false) | Out-Null 
 $r = $ws1.UsedRange
 $r2 = $ws1.Range("B2")
 $r3 = $ws1.Range("E2")
 $r4 = $ws1.Range("C2") 
 $a = $r.Sort($r2,$xlDescending,$r3,$null,$xlAscending, `
 $r4,$xlAscending,$xlYes) 
 $ws1.Application.ActiveWindow.FreezePanes=$true
 [void]$ws1.cells.entireColumn.Autofit()
 $s = $xl.Range("A1").EntireColumn
 $s.Hidden = $true
 $ws1.Range("B2").Select() | Out-Null
 }
 
 
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $row = 1
 $i = 1
 For($i = 1; $i -lt 57; $i++){
 $ws.Cells.Item($row, 1) = "'$'ws.Cells.Item($row, 2).Font.ColorIndex = " + $row
 $ws.Cells.Item($row, 2).Font.ColorIndex = $row
 $ws.Cells.Item($row, 2) = "test " + $row
 $row++
 }
 [void]$ws.cells.entireColumn.Autofit() 
  
  
 function Release-Ref ($info) {
     foreach ( $p in $args ) {
         ([System.Runtime.InteropServices.Marshal]::ReleaseComObject(
         [System.__ComObject]$p) -gt 0)
         [System.GC]::Collect()
         [System.GC]::WaitForPendingFinalizers()
     }
 
 Function XLHyperlinks {
 $link = "http://www.microsoft.com/technet/scriptcenter" 
 $xl = new-object -comobject excel.application
 $xl.Visible = $true
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1) 
 $ws.Cells.Item(1, 1).Value() = "Script Center"
 $r = $ws.Range("A1")
 $objLink = $ws.Hyperlinks.Add($r, $link)
 $a = Release-Ref $r $ws $wb $xl
 
 #
 #
 
 Function xlSum {
 $range = $ws1.usedRange
 $Sumrow = $row + 1
 $functions = $xl.WorkSheetfunction
 $ws1.cells.item($Sumrow,1).Select()
 [void]$range.entireColumn.Autofit()
 
 #
 #
 #
 #
 $xlSum = -4157
 $range = $xl.range("A1:D6")
 $range.Subtotal(1,-4157,(2,3,4),$true,$False,$true) 
 
 #
 
 Function XLSubtotals {
 $xlExpression = 2
 $xlPasteFormats = -4122
 $xlNone = -4142
 $xlToRight = -4161
 $xlToLeft = -4159
 $xlDown = -4121
 $xlShiftToRight = -4161
 $xlFillDefault = 0
 $xlSummaryAbove = 0
 $xlSortValues = $xlPinYin = 1
 $xlAscending = 1
 $xlDescending = 2
 $xlYes = 1
 $xlTopToBottom = 1
 $xlSum = -4157 
 $xl = New-Object -comobject Excel.Application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $ws = $wb.Sheets.Add()
 $xl.Worksheets.item(1).name = "Detail"
 $xl.Worksheets.item(2).name = "ShowLevels1"
 $xl.Worksheets.item(3).name = "ShowLevels2"
 $xl.Worksheets.item(4).name = "ShowLevels3" 
 $ws1 = $wb.worksheets | where {$_.name -eq "Detail"}
 $ws2 = $wb.worksheets | where {$_.name -eq "ShowLevels1"} #<------- Selects sheet 6
 $ws3 = $wb.worksheets | where {$_.name -eq "ShowLevels2"} #<------- Selects sheet 5
 $ws4 = $wb.worksheets | where {$_.name -eq "ShowLevels3"} #<------- Selects sheet 4 
 $ws1.Tab.ColorIndex = 8
 $ws1.Tab.ColorIndex = 5
 $ws2.Tab.ColorIndex = 6
 $ws3.Tab.ColorIndex = 7 
 $ws1.Cells.Item(1,2) = "Amount"
 $ws1.Cells.Item(1,1) = "SalesPerson"
 $ws1.Cells.Item(2,2) = 7324
 $ws1.Cells.Item(2,1) = "Jack"
 $ws1.Cells.Item(3,2) = 294
 $ws1.Cells.Item(3,1) = "Elizabeth"
 $ws1.Cells.Item(4,2) = 41472
 $ws1.Cells.Item(4,1) = "Renee"
 $ws1.Cells.Item(5,2) = 25406
 $ws1.Cells.Item(5,1) = "Elizabeth"
 $ws1.Cells.Item(6,2) = 20480
 $ws1.Cells.Item(6,1) = "Jack"
 $ws1.Cells.Item(7,2) = 11294
 $ws1.Cells.Item(7,1)= "Renee"
 $ws1.Cells.Item(8,2) = 982040
 $ws1.Cells.Item(8,1) = "Elizabeth"
 $ws1.Cells.Item(9,2) = 2622368
 $ws1.Cells.Item(9,1) = "Jack"
 $ws1.Cells.Item(10,2) = 884144
 $ws1.Cells.Item(10,1) = "Renee" 
 $ws1.Range("B2").Select() | Out-Null
 $ws1.Application.ActiveWindow.FreezePanes=$true
 [void]$ws1.cells.entireColumn.Autofit() 
 $ws1.Range("A1").Select() | Out-Null
 $r = $ws.Range("A2:A10")
 $a = $r.sort($r2, $xlAscending) 
 $range1 = $ws1.Range("A1:B1").EntireColumn
 $Range1.Select() | Out-Null
 #$ws.Range.SpecialCells(11)).Select()
 $Range1.Copy()
 $ws1.Range("A1").Select() | Out-Null 
 $ws2.Select() | Out-Null
 [void]$ws2.Range("A1").PasteSpecial(-4163)
 $ws3.Select() | Out-Null
 [void]$ws3.Range("A1").PasteSpecial(-4163)
 $ws4.Select() | Out-Null
 [void]$ws4.Range("A1").PasteSpecial(-4163) 
 $ws2.Select() | Out-Null
 $range = $xl.range("A1:B10")
 $range.Subtotal(1,-4157,(2),$true,$False,$true)
 $ws2.Outline.ShowLevels(1)
 [void]$ws1.cells.entireColumn.Autofit()
 $ws2.Range("A1").Select() | Out-Null 
 $ws3.Select() | Out-Null
 $range = $xl.range("A1:B10")
 $range.Subtotal(1,-4157,(2),$true,$False,$true)
 $ws3.Outline.ShowLevels(2)
 [void]$ws1.cells.entireColumn.Autofit()
 $ws3.Range("A1").Select() | Out-Null 
 $ws4.Select() | Out-Null
 $range = $xl.range("A1:B10")
 $range.Subtotal(1,-4157,(2),$true,$False,$true)
 $ws4.Outline.ShowLevels(3) 
 $ws1.Select() | Out-Null
 [void]$ws1.cells.entireColumn.Autofit()
 $ws1.Range("A1").Select() | Out-Null
 
 #---------------------------------------------------
 #
  
 Function XLAutoFilter {
 $xlTop10Items = 3
 $xlTop10Percent = 5
 $xlBottom10Percent = 6
 $xlBottom10Items = 4
 $xlAnd = 1
 $xlOr = 2
 $xlNormal = -4143
 $xl = New-Object -comobject Excel.Application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 $ws = $wb.Sheets.Add()
 $ws = $wb.Sheets.Add() 
 $ws1 = $wb.worksheets | where {$_.name -eq "Sheet1"}        #<------- Selects sheet 1
 $ws2 = $wb.worksheets | where {$_.name -eq "Sheet2"}         #<------- Selects sheet 2
 $ws3 = $wb.worksheets | where {$_.name -eq "Sheet3"}         #<------- Selects sheet 3
 $ws4 = $wb.worksheets | where {$_.name -eq "Sheet4"}         #<------- Selects sheet 4
 $ws5 = $wb.worksheets | where {$_.name -eq "Sheet5"}        #<------- Selects sheet 5 
 $ws1.Tab.ColorIndex = 8
 $ws2.Tab.ColorIndex = 7
 $ws3.Tab.ColorIndex = 6
 $ws4.Tab.ColorIndex = 5
 $ws5.Tab.ColorIndex = 4 
 $ws1.name = "Detail"
 $ws2.name = "JackOnly"
 $ws3.name = "Top2"
 $ws4.name = "LowestHighest"
 $ws5.name = "Top25Percent" 
 $ws1.cells.Item(1,1) =  "Amount"
 $ws1.cells.Item(1,2) =  "SalesPerson"
 $ws1.cells.Item(2,1) = 1
 $ws1.cells.Item(2,2) = "Jack"
 $ws1.cells.Item(3,1) = 2
 $ws1.cells.Item(3,2) = "Elizabeth"
 $ws1.cells.Item(4,1) = 3
 $ws1.cells.Item(4,2) = "Renee"
 $ws1.cells.Item(5,1) = 4
 $ws1.cells.Item(5,2) = "Elizabeth"
 $ws1.cells.Item(6,1) = 5
 $ws1.cells.Item(6,2) = "Jack"
 $ws1.cells.Item(7,1) = 6
 $ws1.cells.Item(7,2) = "Renee"
 $ws1.cells.Item(8,1) = 7
 $ws1.cells.Item(8,2) = "Elizabeth"
 $ws1.cells.Item(9,1) = 8
 $ws1.cells.Item(9,2) = "Jack"
 $ws1.cells.Item(10,1) = 9
 $ws1.cells.Item(10,2) = "Renee"
 $ws1.cells.Item(11,1) = 10
 $ws1.cells.Item(11,2) = "Jack"
 $ws1.cells.Item(12,1) = 11
 $ws1.cells.Item(12,2) = "Jack"
 $ws1.cells.Item(13,1) = 12
 $ws1.cells.Item(13,2) = "Elizabeth"
 $ws1.cells.Item(14,1) = 13
 $ws1.cells.Item(14,2) = "Renee"
 $ws1.cells.Item(15,1) = 14
 $ws1.cells.Item(15,2) = "Elizabeth"
 $ws1.cells.Item(16,1) = 15
 $ws1.cells.Item(16,2) = "Jack"
 $ws1.cells.Item(17,1) = 16
 $ws1.cells.Item(17,2) = "Renee"
 $ws1.cells.Item(18,1) = 17
 $ws1.cells.Item(18,2) = "Elizabeth"
 $ws1.cells.Item(19,1) = 18
 $ws1.cells.Item(19,2) = "Jack"
 $ws1.cells.Item(20,1) = 19
 $ws1.cells.Item(20,2) = "Renee"
 $ws1.cells.Item(21,1) = 20
 $ws1.cells.Item(21,2) = "Renee" 
 $used = $ws1.usedRange
 $lastCell = $used.SpecialCells($xlCellTypeLastCell)
 $lastrow = $lastCell.row 
 $r = $ws1.Range("A1:B$lastrow")
 $ws1.Range("A1:B$lastrow").Copy() 
 $ws2.Select() | Out-Null
 [void]$ws2.Range("A1").PasteSpecial(-4163)
 $ws3.Select() | Out-Null
 [void]$ws3.Range("A1").PasteSpecial(-4163)
 $ws4.Select() | Out-Null
 [void]$ws4.Range("A1").PasteSpecial(-4163)
 $ws5.Select() | Out-Null
 [void]$ws5.Range("A1").PasteSpecial(-4163)
 #
 $ws5.Range("A1").Select()
 $xl.Range("A1").Select() | Out-Null
 $ws5.cells.Item.EntireColumn.AutoFit 
 $ws2.Select()
 $ws2.Range("A1").Select()
 $ws2.cells.Item.EntireColumn.AutoFit 
 $ws4.Select()
 $ws4.Range("A1").Select()
 $ws4.cells.Item.EntireColumn.AutoFit
 $ws5.Select()
 $ws5.Range("A1").Select()
 $xl.Range("A1").Select() | Out-Null
 $ws5.cells.Item.EntireColumn.AutoFit 
 
 #---------------------------------------------------
 #
 
 #
  
 Function XLFormula1 {
 $xl = New-Object -comobject excel.application
 $xl.Visible = $true
 $xl.DisplayAlerts = $False 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1) 
 $ws.name = "ComplexFormula"
 $ws.Tab.ColorIndex = 9
 $row = 2
 $lastrow = $row
 $Col = 3
 $Off = $Col - 1
 $ws.Cells.Item(1, 1) = "FileName"
 $ws.Cells.Item(1, 2) = "Folder"
 $ws.Cells.Item(1, 3) = "FullPath"
 $ws.Cells.Item(2, 3) = "c:\Folder1\FunctionFolder1\FunctionFolder2\File1.txt"
 $ws.Cells.Item(3, 3) = "c:\Folder1\FunctionFolder1\FunctionFolder2\FunctionFolder3\File2.txt"
 $ws.Cells.Item(4, 3) = "c:\Folder1\FunctionFolder1\FunctionFolder2\FunctionFolder3\FunctionFolder4\File3.txt"
 $lastrow = 4 
 $Range1 = $ws.Range("A2")
 $ws.Cells.Item(2,1).FormulaR1C1 = "=MID(C[2],FIND(CHAR(127),Substitute(C[2],""\"",CHAR(127),LEN(C[2])-LEN(Substitute(C[2],""\"",""""))))+1,254)"
 $range2 = $ws.Range("A2:A$lastrow")
 [void]$range1.AutoFill($range2,$xlFillDefault) 
 $lastrow = 4
 $ws.Range("B2").Select() | Out-Null
 $ws.Cells.Item(2,2).FormulaR1C1 = "=LEFT(C[1],FIND(CHAR(127),Substitute(C[1],""\"",CHAR(127),LEN(C[1])-LEN(Substitute(C[1],""\"",""""))))-1)"
 $R1 = $ws.Range("B2")
 $r2 = $ws.Range("B2:B$lastrow")
 [void]$R1.AutoFill($R2,$xlFillDefault)
 $ws.Cells.Item.EntireColumn.AutoFit
 $ws.Range("A2").Select() | Out-Null
 
 #---------------------------------------------------
 #
 
 #
 Function Pivot {
 $xlPivotTableVersion12     = 3
 $xlPivotTableVersion10     = 1
 $xlCount                 = -4112
 $xlDescending             = 2
 $xlDatabase                = 1
 $xlHidden                  = 0
 $xlRowField                = 1
 $xlColumnField             = 2
 $xlPageField               = 3
 $xlDataField               = 4    
 $PivotTable = $wb.PivotCaches().Create($xlDatabase,"Report!R1C1:R65536C5",$xlPivotTableVersion10)
 $PivotTable.CreatePivotTable("Pivot!R1C1") | Out-Null 
 [void]$ws3.Select()
 $ws3.Cells.Item(1,1).Select()
 $wb.ShowPivotTableFieldList = $true 
 $PivotFields.Orientation = $xlRowField
 $PivotFields.Position = 1 
 $PivotFields.Orientation = $xlColumnField
 $PivotFields.Position = 1 
 $PivotFields.Orientation=$xlDataField 
 $mainRng = $ws3.UsedRange.Cells 
 $RowCount = $mainRng.Rows.Count  
 $R = $RowCount
 $R = $R - 1    
 $mainRng.Select()
 $objSearch = $mainRng.Find("Grand Total")
 $objSearch.Select()
 $C = $objSearch.Column
 $xlSummaryAbove = 0 
 $xlSortValues = $xlPinYin = 1
 $xlAscending = 1 
 $xlDescending = 2 
 $range1 = $ws3.UsedRange 
 $range2 = $ws3.Cells.Item(3, $C) 
`

