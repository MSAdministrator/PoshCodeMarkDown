---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2961
Published Date: 2011-09-21t09
Archived Date: 2011-11-05t17
---

# get-windowsexperience - 

## Description

get the windows experience rating

## Comments



## Usage



## TODO



## function

`get-windowsexperiencerating`

## Code

`#
 #
 function Get-WindowsExperienceRating {
 #.Synopsis
 #.Parameter ComputerName
 [CmdletBinding()]
 param(
   [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
   [string[]]$ComputerName='localhost'
 )
 process {
   Get-WmiObject -ComputerName $ComputerName Win32_WinSAT | ForEach-Object {
      if($_.WinSatAssessmentState -ne 1) {
         Write-Warning "WinSAT data for '$($_.__SERVER)' is out of date ($($_.WinSatAssessmentState))"
      }
      $_
   } | Select-Object -Property @{name="Computer";expression={$_.__SERVER}}, *Score #, WinSPRLevel
 }
 }
`

