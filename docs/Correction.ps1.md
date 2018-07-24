---
Author: klaus schulte
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5750
Published Date: 2015-02-21t15
Archived Date: 2015-04-24t02
---

# correction - 

## Description

$menu = "

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $menu = @"
 =================================
       conference 2015 book sales
 =================================
     1. Book Title
     2. Book discription
     3. book id
     4. Book List Price
     5. display Sales Info
     6. Write to File
     7. Exit Program
 ==================================
 "@
 
 switch ($responce)
 {
     1 { [string] $title=read-host "enter title"
             if ($title -match '\w') {continue}
                 else {write-warning "`@ enter a valid title..."
                 pause
                 break
                 }
`

