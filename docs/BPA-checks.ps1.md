---
Author: pwilkinson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5261
Published Date: 2016-06-25t12
Archived Date: 2016-04-15t18
---

# bpa checks - 

## Description

get what windows features are installed, check if there is a best practice for it available, run the bpa and filter results for anything that isn’t informational and dump it to text file

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-WindowsFeature | where {$_.Installed -eq "Installed"} | where {$_.bestPracticesModelId -like "micro*"} | foreach { $_.BestPracticesModelId } | get-bpamodel | invoke-bpamodel
 Get-WindowsFeature | where {$_.Installed -eq "Installed"} | where {$_.bestPracticesModelId -like "micro*"} | foreach { $_.BestPracticesModelId } | get-bpamodel | get-bparesult | Where { $_.Severity -ne "Information" } | out-file c:\BPA-All.txt -encoding ascii
`

