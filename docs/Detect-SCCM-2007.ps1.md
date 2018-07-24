---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3555
Published Date: 2012-08-01t12
Archived Date: 2012-08-04t23
---

# detect sccm 2007 - 

## Description

this is a very simple powershell function to test if the sccm 2007 agent is installed on a machine or not.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function global:test-sccmagent {
 param($PC)
 [boolean]$result=get-wmiobject -Query "Select * from win32_service where Name = 'CcmExec'" -ComputerName $PC
 return $result
 }
`

