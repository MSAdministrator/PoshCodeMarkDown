---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 885
Published Date: 2009-02-20t22
Archived Date: 2016-08-09t21
---

# pivot-object - 

## Description

takes a series of objects (like the converted input from csv) where there are a series of objects (rows) that actually define the same object, and there is (at least) one property which is a unique identifier (and appears on each object), and there are two properties which are a name and value pair defining additional properties ... and outputs new objects which merge the objects and add the name-value pairs as new properties.

## Comments



## Usage



## TODO



## script

`pivot-object`

## Code

`#
 #
 #.Synopsis
 #.Description
 #
 #.Parameter GroupOnColumn
 #.Parameter NameColumn
 #.Parameter ValueColumn
 #.Parameter InputObject
 #.Parameter Jagged
 #.Example
 #
 #
 #.Example
 #@"
 #"@.Split("`n") | ConvertFrom-Csv | Pivot-Objects SamAccountName Attribute Value
 #
 #
 #
 #.Notes
 #
 #
 PARAM(
    [string]$GroupOnColumn
 ,  [string]$NameColumn
 ,  [string]$ValueColumn
 ,  [object[]]$InputObject=@()
 ,  [switch]$Jagged
 )
 PROCESS {
    if($_) {
       $InputObject += $_
    }
 }
 END {
    if($InputObject[0] -isnot [PSObject]) { 
             $first = new-object PSObject $first  
    } else { $first = $InputObject[0]   }
 
    [string[]]$extra = @( $first.PSObject.Properties | 
                          &{PROCESS{$_.Name}} | 
                          Where { ($columns -notcontains $_) -and 
                                  (@($NameColumn,$ValueColumn) -notContains $_)
                          } )
 
    [string[]]$columns = @($GroupOnColumn) + $extra +
                         @($InputObject | Select-Object -Expand $NameColumn | Sort-Object -Unique)
 
    ForEach($item in  $InputObject | group-Object $GroupOnColumn) {
       $thing = New-Object PSObject | 
                Add-Member -Type NoteProperty -Name $GroupOnColumn -Value $item.Name -passthru
 
       foreach($name in $extra) {
          Add-Member -input $thing -MemberType NoteProperty -Name $name -Value $item.Group[0].$name
       }
 
       foreach($i in $item.Group) { 
          Add-Member -Input $thing -Type NoteProperty -Name $i.$NameColumn -Value $i.$ValueColumn 
       }
       if($Jagged) {
          Write-Output $thing
       } else {
          Write-Output $thing | select $columns
       }
    }
    
 }
 #}
`

