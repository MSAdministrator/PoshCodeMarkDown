---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1942
Published Date: 2011-06-29t21
Archived Date: 2016-03-06t23
---

# assert - 

## Description

a simple “assert” for testing

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Assert {
 #.Example
 #
 [CmdletBinding()]
 param( 
    [Parameter(Position=0,ParameterSetName="Script",Mandatory=$true)]
    [ScriptBlock]$condition
 ,
    [Parameter(Position=0,ParameterSetName="Bool",Mandatory=$true)]
    [bool]$success
 ,
    [Parameter(Position=1,Mandatory=$true)]
    [string]$message
 )
 
    $message = "ASSERT FAILED: $message"
   
    if($PSCmdlet.ParameterSetName -eq "Script") {
       try {
          $ErrorActionPreference = "STOP"
          $success = &$condition
       } catch {
          $success = $false
          $message = "$message`nEXCEPTION THROWN: $($_.Exception.GetType().FullName)"         
       }
    }
    if(!$success) {
       throw $message
    }
 }
`

