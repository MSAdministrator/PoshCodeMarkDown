---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2262
Published Date: 
Archived Date: 2010-09-26t20
---

# new-fulldataset - 

## Description

generates a simple dataset based on the files in the current directory.  i whipped this up while answering questions about databinding because i needed to be able to create multiple datasets/datatables easily, without needing an actual database.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $global:dt1 = New-Object system.data.datatable "Times"
 $global:dt2 = New-Object system.data.datatable "Properties"
 $global:ds = New-Object system.data.dataset "dataset"
 $null = $ds.Tables.Add( $dt1 )
 $null = $ds.Tables.Add( $dt2 )
 
 $global:cols1 = ls|?{!$_.PsIsContainer}| gm -type Properties "Name", "CreationTime", "LastAccessTime", "LastWriteTime"
 $global:cols2 = ls|?{!$_.PsIsContainer}| gm -type Properties "Name", "Length", "Extension", "Mode", "FullName"
 
 foreach($c in $cols1){
 	$null = $dt1.Columns.Add( $c.Name, $($c.Definition -split " ")[0] )
 }
 foreach($c in $cols2){
 	$null = $dt2.Columns.Add( $c.Name, $($c.Definition -split " ")[0] )
 }
 
 foreach($f in ls|?{!$_.PsIsContainer}){ 
 	$null = $dt1.LoadDataRow( @($cols1 | % { $f.$($_.Name) }), $null )
 	$null = $dt2.LoadDataRow( @($cols2 | % { $f.$($_.Name) }), $null )
 }
 
 return $ds
 
`

