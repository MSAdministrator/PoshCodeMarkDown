---
Author: cory delamarter (increased speed)
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 644
Published Date: 
Archived Date: 2008-10-27t18
---

# list object discoveries - 

## Description

enumerate opsmgr 2007 object discoveries targeted to windows server

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 get-discovery | ? {$_.Target -match $(get-monitoringclass -Name "Microsoft.Windows.Server.Computer").Id} | ft Name, DisplayName
`

