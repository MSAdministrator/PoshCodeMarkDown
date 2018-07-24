---
Author: alanrenouf
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 490
Published Date: 2009-07-30t04
Archived Date: 2012-01-02t09
---

# quickstats from vmware - 

## Description

quickly create a jpg barchart of vmware stats which can be adjusted in the variables, office components will be needed.  for more examples and comments please check http

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 connect-VIServer yourserver
 
 
 $Caption = "CPU Usage"
 $Stat = "cpu.usage.average"
 $NumToReturn = 20
 
 $categories = @()
 $values = @()
 $chart = new-object -com OWC11.ChartSpace.11
 $chart.Clear()
 $c = $chart.charts.Add(0)
 $c.Type = 4
 $c.HasTitle = "True"
 $series = ([array] $chart.charts)[0].SeriesCollection.Add(0)
 
 Get-Stat -Entity (Get-vm) -Stat $stat -MaxSamples 1 -Realtime |Sort-Object Value -Descending | Select-Object Entity, Value -First $NumToReturn | foreach-object {
 
 	$categories += $_.Entity.Name
 	$values += $_.Value * 1
 }
 
 $series.Caption = $Caption
 $series.SetData(1, -1, $categories)
 $series.SetData(2, -1, $values)
 $filename = (resolve-path .).Path + "\Chart.jpg"
 $chart.ExportPicture($filename, "jpg", 800, 500)
 invoke-item $filename
`

