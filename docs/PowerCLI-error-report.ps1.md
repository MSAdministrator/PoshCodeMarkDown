---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 2020
Published Date: 
Archived Date: 2010-07-26t15
---

# powercli error report - 

## Description

generating error report bundle for vmware powercli / vsphere

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $getVmScript = { 
 	Connect-VIServer yourVCenterServer
 	Get-VM
 }
 $ getVmScript | Get-ErrorReport -ProblemScriptTimeoutSeconds 60 -ProblemDescription "Get-VM hangs when trying to retrieve all the VMs form the server. The server�s inventory can be successfully browsed via the vClient." -Destination 'D:\bug report'
`

