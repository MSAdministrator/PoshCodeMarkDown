---
Author: david sjstrand
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6378
Published Date: 2016-06-11t14
Archived Date: 2016-06-16t09
---

# resultantsetofpolicy - 

## Description

the get-gpresultantsetofpolicy requires the -path parameter and saves the output to file. this command should have been named export-gpresultantsetofpolicy. this is a script that does what a command with the verb get should do; return the result to the pipeline.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 param([String]$Computer = 'localhost', [String]$User, [Microsoft.GroupPolicy.ReportType]$ReportType = 'XML')
 $gprsop = new-object Microsoft.GroupPolicy.GPRsop([Microsoft.GroupPolicy.RsopMode]::Logging,'')
 $gprsop.LoggingComputer = $Computer
 if ($PSBoundParameters.ContainsKey('User')) {
 	$gprsop.LoggingUser = $User
 }
 $gprsop.CreateQueryResults()
 $report = $gprsop.GenerateReport($ReportType)
 if ($ReportType -eq 'XML') {
     return [xml]$report
 }
 return $report
`

