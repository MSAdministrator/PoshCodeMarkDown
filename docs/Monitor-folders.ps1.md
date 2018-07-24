---
Author: dfsdiag
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6042
Published Date: 2016-10-08t10
Archived Date: 2016-05-17t12
---

# monitor folders - 

## Description

compare files in multiple folders against a reference set to provide an early detection of cryptolocker

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $monitoredServers = gc "C:\ServersToMonitor.txt"
 $referenceDirectory = "C:\CLEAN\"
 
 $monitoredFiles = gci -Recurse $referenceDirectory | ? { ! $_.PSIsContainer } | % { $($_.FullName).replace("$referenceDirectory","") }
 
 foreach ($server in $monitoredServers) {
     Write-Host "Checking: $server" -ForegroundColor GREEN
     
     foreach ($file in $monitoredFiles) {
         Write-Host "Checking: \\$server\$file against $referenceDirectory\$file"
         
         if ((!(Test-Path "\\$server\$file")) -or (Compare-Object $(gc "\\$server\$file") $(gc $referenceDirectory\$file))) {
                Write-Host "FILES DO NOT MATCH" -ForegroundColor RED
         }
     }
 }
`

