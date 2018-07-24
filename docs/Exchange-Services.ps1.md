---
Author: littlegun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3639
Published Date: 2012-09-14t11
Archived Date: 2012-09-21t03
---

# exchange services - 

## Description

this script checks services on exchange servers and restarts them if needed.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Server = read-host "Enter Exchange Server Name"
 
 $Status = (Get-Service -ComputerName $server |Where-object {$_.Displayname -like "*Exchange*"})
 
   foreach ($Name in $status) {
      IF ($Name.status -eq "Stopped"){Write-Host $Name.displayname "was stopped" -foreground red}} 
 
   foreach ($Name in $status) {
      IF ($Name.status -eq "Stopped"){Start-Service -InputObject $Name}}
 
 Get-Service -ComputerName $server |Where-object {$_.Displayname -like "*Exchange*"
 }
`

