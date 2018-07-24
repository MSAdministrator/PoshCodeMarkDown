---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 165
Published Date: 2008-04-03t10
Archived Date: 2017-04-30t13
---

# start-elevated - 

## Description

a simple function based on http

## Comments



## Usage



## TODO



## function

`start-elevated`

## Code

`#
 #
    param ($app) 
    $psi = new-object "System.Diagnostics.ProcessStartInfo"
    $psi.FileName = $app; 
    $psi.Arguments = [string]$args; 
    $psi.Verb = "runas";
    [System.Diagnostics.Process]::Start($psi)
 #}
`

