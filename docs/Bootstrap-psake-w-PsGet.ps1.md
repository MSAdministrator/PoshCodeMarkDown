---
Author: codemonkeyking
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3876
Published Date: 2013-01-11t16
Archived Date: 2013-01-22t13
---

# bootstrap psake w/ psget - 

## Description

bootstrap powershell psake and ‘patch’ it if necessary. use and modify as desired.

## Comments



## Usage



## TODO



## module

`get-mymodule`

## Code

`#
 #
 function Get-MyModule {
     param ([string] $name)
     write-host "Is '$name' already imported? " -NoNewline -ForegroundColor Cyan
     if (-not(Get-Module -name $name)) {
         write-host "No." -ForegroundColor Red
         write-host "Can we import '$name'? " -NoNewline
         if (Get-Module -ListAvailable | Where-Object {$_.Name -eq $name}) {
             write-host "Yes. Importing '$name'." -ForegroundColor Green
             import-module -name $name
             return $true
         }
         else {
             write-host "No. It is not available locally." -ForegroundColor Cyan
             return $false
         }
     }
     else {
         write-host "Yes." -ForegroundColor Green
         return $true
     }
 }
 
 function ContainsAny( [string]$s, [string[]] $items) {
     $matches = @($items | where { $_.Contains( $s ) })
     return [bool]$matches
 }
 
 if (-not(Get-MyModule -name psake)) {
     if (-not(Get-MyModule -name PsGet)) {
         write-host "Downloading and installing PsGet from repository..." -ForegroundColor Yellow
         (new-object Net.WebClient).DownloadString("http://psget.net/GetPsGet.ps1") | iex
         import-module PsGet
     }
 
     write-host "Installing psake..." -ForegroundColor Yellow
     install-module psake
 }
 
 import-module psake
 
 $output = invoke-psake
 if (ContainsAny "Cannot bind argument to parameter 'Path' because it is an empty string." $output) {
     write-host "We need to patch psake.psm1... " -ForegroundColor Red -NoNewline
     $psakePath = "~\Documents\WindowsPowerShell\Modules\psake\psake.psm1"
     (get-content $psakePath) | ForEach-Object { return $_.Replace("[Parameter(Position = 0, Mandatory = 0)][string] `$buildFile,", "[Parameter(Position = 0, Mandatory = 0)][string] `$buildFile = `"default.ps1`",")} | Set-Content $psakePath
     import-module psake -Force
 
     $output = invoke-psake
     if (ContainsAny "Cannot bind argument to parameter 'Path' because it is an empty string." $output) {
         write-host "patch was unsuccessful!" -ForegroundColor Red
     }
     else {
         write-host "patch was successfully applied." -ForegroundColor Green
     }
 }
 
 write-host "Invoke-Psake available as alias 'psake'" -ForegroundColor Cyan
 new-alias psake invoke-psake -Scope "Global"
`

