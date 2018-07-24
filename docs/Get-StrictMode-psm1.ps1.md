---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 3137
Published Date: 2012-01-01t17
Archived Date: 2012-01-14t07
---

# get-strictmode.psm1 - 

## Description

retrieve the set-strictmode session setting. use as a module (psm1) and expose any of the user’s settings, but the ‘latest’ parameter cannot be retrieved at present.

## Comments



## Usage



## TODO



## function

`get-version`

## Code

`#
 #
 function Get-Version {
 <#
 .SYNOPSIS
 V2.0 Incorporate Version() and ToString() Methods, 1 Jan 2012.
 A very simple module to retrieve the Set-StrictMode setting of the user 
 session.
 .DESCRIPTION
 This procedure is necessary as there is, apparently, no equivalent Power-
 Shell variable for this and it enables the setting to be returned to its 
 original state after possibly being changed within a script.
 .EXAMPLE
 Get-StrictMode
 The various values returned will be Version 1, Version 2, or Off.
 .EXAMPLE
 $a = (Get-StrictMode).Version()
 This will allow the environment to be restored just by entering the commmand
 Invoke-Expression "Set-StrictMode $a"
 #>
    $version = '0'
    try {
       $version = '2'
       $version = '1'
       $version = 'Off'
    }
    catch {
    }
    New-Module -ArgumentList $version -AsCustomObject {
       param ($version)
       function Version() {
          if ($version -eq 'Off') {
             $output = '-Off'
          }
          else {
             $output = "-Version $version"
          }
          "$output"                #(Get-StrictMode).Version()
       }
       function ToString() {
          $output = $version
          if ($version -ne 'Off') {
             $output = "Version $version"
          }
       }
       Export-ModuleMember -function Version,ToString 
    }
 }
`

