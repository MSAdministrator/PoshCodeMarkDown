---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 5734
Published Date: 2016-02-12t13
Archived Date: 2016-04-27t22
---

# librarychart - 

## Description

defines functions for wokring with  microsoft chart control for .net 3.5 framework.pipe output to out-chart function and specify chart type. chart will display in form or save to image file. real-time charts are supported by passing in a script block. updated to fix line chart. line chart xaxis needs tweaking.

## Comments



## Usage



## TODO



## script

`new-chart`

## Code

`#
 #
 ###
 ###
 ###
 ###
 ###
 [void][Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
 [void][Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms.DataVisualization")
 
 #######################
 function New-Chart
 {
     param ([int]$width,[int]$height,[int]$left,[int]$top,$chartTitle)
     $global:Chart = New-object System.Windows.Forms.DataVisualization.Charting.Chart 
     $global:Chart.Width = $width 
     $global:Chart.Height = $height 
     $global:Chart.Left = $left 
     $global:Chart.Top = $top
     $chartArea = New-Object System.Windows.Forms.DataVisualization.Charting.ChartArea 
     $global:chart.ChartAreas.Add($chartArea)
 
     [void]$global:Chart.Titles.Add($chartTitle) 
 
     $global:Chart.BackColor = [System.Drawing.Color]::Transparent
 
 
 #######################
 function New-BarColumnChart
 {
     param ([hashtable]$ht, $chartType='Column', $chartTitle,$xTitle,$yTitle, [int]$width,[int]$height,[int]$left,[int]$top,[bool]$asc)
 
     New-Chart -width $width -height $height -left $left -top $top -chartTitle $chartTitle
 
     $chart.ChartAreas[0].AxisX.Title = $xTitle
     $chart.ChartAreas[0].AxisY.Title = $yTitle
 
     [void]$global:Chart.Series.Add("Data")
     $global:Chart.Series["Data"].Points.DataBindXY($ht.Keys, $ht.Values)
 
     $global:Chart.Series["Data"].ChartType = [System.Windows.Forms.DataVisualization.Charting.SeriesChartType]::$chartType
 
     if ($asc)
     { $global:Chart.Series["Data"].Sort([System.Windows.Forms.DataVisualization.Charting.PointSortOrder]::Ascending, "Y") }
     else
     { $global:Chart.Series["Data"].Sort([System.Windows.Forms.DataVisualization.Charting.PointSortOrder]::Descending, "Y") }
     
     $global:Chart.Series["Data"]["DrawingStyle"] = "Cylinder"
     $global:chart.Series["Data"].IsValueShownAsLabel = $true
     $global:chart.Series["Data"]["LabelStyle"] = "Top"
 
 
 
 #######################
 function New-LineChart
 {
 
     param ([hashtable]$ht,$chartTitle, [int]$width,[int]$height,[int]$left,[int]$top)
 
     New-Chart -width $width -height $height -left $left -top $top -chartTitle $chartTitle
 
     [void]$global:Chart.Series.Add("Data")
     $global:Chart.Series["Data"].Points.DataBindXY($ht.Keys,$ht.Values)
 
     #$global:Chart.Series["Data"].XValueType = [System.Windows.Forms.DataVisualization.Charting.ChartValueType]::Time 
     #$global:Chart.chartAreas[0].AxisX.LabelStyle.Format = "hh:mm:ss"
     #$global:Chart.chartAreas[0].AxisX.LabelStyle.Interval = 1
     #$global:Chart.chartAreas[0].AxisX.LabelStyle.IntervalType = [System.Windows.Forms.DataVisualization.Charting.DateTimeIntervalType]::Seconds 
     $global:Chart.Series["Data"].ChartType = [System.Windows.Forms.DataVisualization.Charting.SeriesChartType]::Line
     #$global:chart.Series["Data"].IsValueShownAsLabel = $false
 
 
 #######################
 function New-PieChart
 {
 
     param ([hashtable]$ht,$chartTitle, [int]$width,[int]$height,[int]$left,[int]$top)
 
     New-Chart -width $width -height $height -left $left -top $top -chartTitle $chartTitle
 
     [void]$global:Chart.Series.Add("Data")
     $global:Chart.Series["Data"].Points.DataBindXY($ht.Keys, $ht.Values)
 
     $global:Chart.Series["Data"].ChartType = [System.Windows.Forms.DataVisualization.Charting.SeriesChartType]::Pie
 
     $global:Chart.Series["Data"]["PieLabelStyle"] = "Outside" 
     $global:Chart.Series["Data"]["PieLineColor"] = "Black" 
     #$global:chart.Series["Data"].IsValueShownAsLabel = $true
     #$legend = New-object System.Windows.Forms.DataVisualization.Charting.Legend
     #$global:Chart.Legends.Add($legend)
     #$Legend.Name = "Default"
 
 
 #######################
 function Remove-Points
 {
     param($name='Data',[int]$maxPoints=200)
     
     while ( $global:chart.Series["$name"].Points.Count > $maxPoints )
     { $global:chart.Series["$name"].Points.RemoveAT(0) }
 
 
 #######################
 function Out-Form
 {
     param($interval,$scriptBlock,$xField,$yField)
 
     $global:Chart.Anchor = [System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Right -bor 
                     [System.Windows.Forms.AnchorStyles]::Top -bor [System.Windows.Forms.AnchorStyles]::Left 
     $Form = New-Object Windows.Forms.Form 
     $Form.Text = 'PowerCharts'
     $Form.Width = 600
     $Form.Height = 600 
     $Form.controls.add($global:Chart)
     if ($scriptBlock -is [ScriptBlock])
     { 
         if (!($xField -or $yField))
         { throw 'xField and yField required with scriptBlock parameter.' }
         $timer = New-Object System.Windows.Forms.Timer 
         $timer.Interval = $interval
         $timer.add_Tick({
  
         $ht = &$scriptBlock | ConvertTo-HashTable $xField $yField
         if ($global:Chart.Series["Data"].ChartTypeName -eq 'Line')
         {
             Remove-Points
             $ht | foreach { $global:Chart.Series["Data"].Points.AddXY($($_.Keys), $($_.Values)) }               
         }
         else
         { $global:Chart.Series["Data"].Points.DataBindXY($ht.Keys, $ht.Values) }
         $global:chart.ResetAutoValues()
         $global:chart.Invalidate()
  
         })
         $timer.Enabled = $true
         $timer.Start()
         
     }
     $Form.Add_Shown({$Form.Activate()}) 
     $Form.ShowDialog()
 
 
 #######################
 function Out-ImageFile
 {
     param($fileName,$fileformat)
 
     $global:Chart.SaveImage($fileName,$fileformat)
 }
 #######################
 #######################
 function ConvertTo-Hashtable
 { 
     param([string]$key, $value) 
 
     Begin 
     { 
         $hash = @{} 
     } 
     Process 
     { 
         $thisKey = $_.$Key
         $hash.$thisKey = $_.$Value 
     } 
     End 
     { 
         Write-Output $hash 
     }
 
 
 #######################
 function Out-Chart
 {
     param(  $xField=$(throw 'Out-Chart:xField is required'),
             $yField=$(throw 'Out-Chart:yField is required'), 
             $chartType='Column',$chartTitle,
             [int]$width=500,
             [int]$height=400,
             [int]$left=40,
             [int]$top=30,
             $filename,
             $fileformat='png',
             [int]$interval=1000,
             $scriptBlock,
             [switch]$asc
         )
 
     Begin
     {
         $ht = @{}
     }
     Process
     {
        if ($_)
        {
         $thisKey = $_.$xField
         $ht.$thisKey = $_.$yField 
        }
     }
     End
     {
         if ($scriptBlock)
         { $ht = &$scriptBlock | ConvertTo-HashTable $xField $yField }
         switch ($chartType)
         {
             'Bar' {New-BarColumnChart -ht $ht -chartType $chartType -chartTitle $chartTitle -width $width -height $height -left $left -top $top -asc $($asc.IsPresent)}
             'Column' {New-BarColumnChart -ht $ht -chartType $chartType -chartTitle $chartTitle -width $width -height $height -left $left -top $top -asc $($asc.IsPresent)}
             'Pie' {New-PieChart -chartType -ht $ht  -chartTitle $chartTitle -width $width -height $height -left $left -top $top }
             'Line' {New-LineChart -chartType -ht $ht -chartTitle $chartTitle -width $width -height $height -left $left -top $top }
 
         }
 
         if ($filename)
         { Out-ImageFile $filename $fileformat }
         elseif ($scriptBlock )
         { Out-Form $interval $scriptBlock $xField $yField }
         else
         { Out-Form }
     }
 
`

