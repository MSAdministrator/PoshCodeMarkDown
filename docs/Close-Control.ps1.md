---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4284
Published Date: 2013-07-02t11
Archived Date: 2013-07-09t04
---

# close-control.ps1 - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## function

`close-control`

## Code

`#
 #
 function Close-Control
 {
     param(
     [Parameter(Mandatory=$true,
         ValueFromPipeline=$true)]
     [Windows.Media.Visual]
     $Visual
     )
     
     process {
         if ($Visual -is [Windows.Window]) {
             $Visual.Close()
         }
         if ($Visual.Parent -is [Windows.Window]) {
             $Visual.Close()
         }
         $Visual.Visibility = "Collapsed"
     }
 }
`

