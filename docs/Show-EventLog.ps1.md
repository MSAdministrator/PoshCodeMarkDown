---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4465
Published Date: 2013-09-12t10
Archived Date: 2013-09-17t23
---

# show-eventlog - show-eventlog.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`show-eventlog`

## Code

`#
 #
 
   #############################################################################################
 #
 #
 
 Function Show-EventLog ($Server,$log,$Latest)
 {
 Get-EventLog  -computername $server -log $log -newest $latest | Out-GridView
 }
`

