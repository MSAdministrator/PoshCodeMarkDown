---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 741
Published Date: 
Archived Date: 2008-12-21t05
---

# start-process - 

## Description

this is a simple function that can “start” apps and return the process object.  in particular, it can start uris, documents, and apps defined in the “app paths” registry, and basically anything that you could start from the run dialog.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Start($app,$param) {
    if($param) {
       [Diagnostics.Process]::Start( $app, $param )
    } else {
       [Diagnostics.Process]::Start( $app )
    }
 }
`

