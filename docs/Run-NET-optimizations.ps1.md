---
Author: scott copus
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5959
Published Date: 2016-08-01t14
Archived Date: 2016-05-17t16
---

# run .net optimizations - 

## Description

force .net framework optimization to happen (causes high cpu by mscorsvw.exe)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Remove-Item variable:frameworks
 $frameworks = @("$env:SystemRoot\Microsoft.NET\Framework")
 If (Test-Path "$env:SystemRoot\Microsoft.NET\Framework64") {
     $frameworks += "$env:SystemRoot\Microsoft.NET\Framework64"
 }
 
 ForEach ($framework in $frameworks) {
     $ngen_path = Join-Path (Join-Path $framework -childPath (gci $framework | where { ($_.PSIsContainer) -and (Test-Path (Join-Path $_.FullName -childPath "ngen.exe")) } | Sort Name -Descending | Select -First 1).Name) -childPath "ngen.exe"
     Write-Host "$ngen_path executeQueuedItems"
     Start-Process $ngen_path -ArgumentList "executeQueuedItems" -Wait
 }
`

