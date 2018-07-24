---
Author: thom lamb
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6474
Published Date: 2017-08-13t03
Archived Date: 2017-03-31t03
---

# powergrowl sample - 

## Description

powergrowl sample goes here

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Clear-Host
 $Location = $($Env:PSModulePath).Split(';')[0]
 
 Import-Module $Location\PowerGrowl.psm1
 
 Get-Module PowerGrowl | Format-List
 
 
 Register-GrowlType -AppName "PoshTwitter" -Name "Greetings" `
     -Icon "C:\Users\username\Documents\WindowsPowerShell\Modules\default_icon.png"
 Send-Growl "Greetings" "Hello World!"
`

