---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1679
Published Date: 
Archived Date: 2010-03-06t09
---

# get-vmcreationreport - 

## Description

produces a report of the number of and names of vms created broken down by month and year.

## Comments



## Usage



## TODO



## script

`get-vmcreationreport`

## Code

`#
 #
 function Get-VMCreationReport {
 	Get-VM | Group {
 		if ($_.CustomFields["CreatedOn"] -as [DateTime] -ne $null) {
 			"{0:Y}" -f [DateTime]$_.CustomFields["CreatedOn"]
 		} else {
 			"Unknown"
 		}
 	}
 }
 
`

