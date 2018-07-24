---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6401
Published Date: 2016-06-20t19
Archived Date: 2016-06-25t05
---

# demo-shouldprocess - 

## Description

shows how to use supportsshouldprocess and confirm

## Comments



## Usage



## TODO



## function

`test-confirm`

## Code

`#
 #
 function Test-Confirm {
     [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact="Medium")]
     param([Switch]$Force)
 
     $RejectAll = $false;
     $ConfirmAll = $false;
     Write-Verbose "ConfirmPreference is $ConfirmPreference"
 
     foreach($file in ls) {
         if($PSCmdlet.ShouldProcess( "Removed the file '$($file.Name)'",
                                     "Remove the file '$($file.Name)'?",
                                     "Removing Files" )) {
             if($Force -Or $PSCmdlet.ShouldContinue("Are you REALLY sure you want to remove '$($file.Name)'?", "Removing '$($file.Name)'", [ref]$ConfirmAll, [ref]$RejectAll)) {
                 "Removing $File"
             }
         }
     }
 }
`

