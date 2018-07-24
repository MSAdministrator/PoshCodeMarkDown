---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2390
Published Date: 2011-11-26t02
Archived Date: 2014-08-18t21
---

# compare sql tables ii - 

## Description

another version to compare 2 sql tables. uses code to exclude specified columns from select *

## Comments



## Usage



## TODO



## function

`convert-tabletolist`

## Code

`#
 #
 function Convert-TableToList
 {
     param(
         $t,
         $colid = 0
     )
     $t | % {$_.item($colid)}
 }
 
 
 function Compare-Tables
 {
     param(
         $name,
         $db1,
         $db2,
         $exclude = @()
         )
 
 
 $sql = "select name from sys.columns  where object_id = object_id('$db1..$name') order by column_id"
 Invoke-ExecuteSql  $sql 'variable' columns
 
 $columns = Convert-TableToList $columns | % { if ($exclude -notcontains $_) {$_} }
 $columnlist = $columns -join ', '
 $sql = @"
 Select 1 [table], $columnlist from $db1..$name
 except
 Select 1 [table], $columnlist from $db2..$name
 union
 Select 2 [table], $columnlist from $db2..$name
 except
 Select 2 [table], $columnlist from $db1..$name
 ORDER by 2
 "@
 $sql
 Invoke-ExecuteSql  $sql 'grid'
 }
 
`

