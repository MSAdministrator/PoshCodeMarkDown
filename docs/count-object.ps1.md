---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2533
Published Date: 2011-03-01t14
Archived Date: 2016-09-07t04
---

# count-object - 

## Description

#a function to count how many items there are, whether its an array, a collection, a hashtable or actually just

## Comments



## Usage



## TODO



## function

`count-object`

## Code

`#
 #
 function count-object ($InputObject)
 {
  if ($inputobject -eq $Null ) { return 0}
  if ($inputobject -is [system.array]) { return $inputobject.length }
  if ($inputobject -is [system.collections.ICollection] -or 
      $inputobject -is [system.collections.IList] -or
      $inputobject -is [system.collections.IDictionary] )  { return $inputobject.count }
  if ($inputobject -is [string]) { return 1 }
  #-1 to show that it is enumerable, but we can't know its length, it could be infinate, 
  if ($inputobject -is [system.collections.IEnumerable]) { return -1 }
  return 1
 }
 set-alias count count-object
 count (get-process)
 count (1,2,3)
 count "hello"
 count 3
 count @{first = 1; second = 2 }
 
 $a = new-object system.collections.arraylist
 [void] $a.add(4);
 [void] $a.add("yo");
 
 count $a
`

