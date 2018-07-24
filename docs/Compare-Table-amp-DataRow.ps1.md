---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2389
Published Date: 2011-11-25t04
Archived Date: 2016-10-19t03
---

# compare table &amp; datarow - 

## Description

compare 2 system.data.dataset each containing 1 table

## Comments



## Usage



## TODO



## function

`compare-datarow`

## Code

`#
 #
 function Compare-DataRow
 {
     param( $a, $b)
     
     
     $diff = ''
     $a_columncount = $a.Table.columns.count
     $b_columncount = $b.Table.columns.count
    
     if ( $a_columncount -ne $b_columncount)
     {
         Write-host "Tables have different number of columns"
     }
     foreach ( $i in 0..($a_columncount - 1))
     {
         if ($a.item($i) -ne $b.item($i))
         {
             $diff += ' ' +  $a.item($i) + '  <> ' +  $b.item($i) +';'
         }
     }     
     $diff
 }
 
 function Compare-Table
 {
     param( $a, $b)
     
 
     $diff = ''
     $a_rowcount = $a.Rows.count
     $b_rowcount = $b.Rows.count
    
     if ( $a_rowcount -ne $b_rowcount)
     {
         Write-host "Tables have different number of columns"
     }
     foreach ( $i in 0..($a_rowcount - 1))
     {
          Compare-DataRow $a.rows[$i] $b.rows[$i]
     }     
     $diff
 }
 
 Compare-DataRow $a.tables[0].rows[0] $b.tables[0].rows[0] 
 Compare-Table   ($a.tables[0]) ($b.tables[0])
`

