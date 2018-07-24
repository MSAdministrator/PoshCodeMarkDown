---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2182
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# invoke-longrunningoperat - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Demonstrates the functionality of the Write-Progress cmdlet
 
 #>
 
 Set-StrictMode -Version Latest
 
 $activity = "A long running operation"
 $status = "Initializing"
 
 for($counter = 0; $counter -lt 100; $counter++)
 {
     $currentOperation = "Initializing item $counter"
     Write-Progress $activity $status -PercentComplete $counter `
         -CurrentOperation $currentOperation
     Start-Sleep -m 20
 }
 
 $status = "Running"
 
 for($counter = 0; $counter -lt 100; $counter++)
 {
     $currentOperation = "Running task $counter"
     Write-Progress $activity $status -PercentComplete $counter `
         -CurrentOperation $currentOperation
     Start-Sleep -m 20
 }
`

