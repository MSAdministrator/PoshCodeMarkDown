---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 174
Published Date: 2008-04-12t13
Archived Date: 2017-05-13t16
---

# get-app - 

## Description

attempt to resolve the path to an executable using get-command or the “app paths” registry key — returns an applicationinfo object

## Comments



## Usage



## TODO



## script

`get-app`

## Code

`#
 #
 ##################################################################################################
 ##################################################################################################
 ##################################################################################################
    param( [string]$cmd )
 
    $command = $null
    $eap = $ErrorActionPreference
    $ErrorActionPreference = "SilentlyContinue"
    $command = Get-Command $cmd | Select -First 1
    $ErrorActionPreference = $eap
   
    if($command -eq $null) {
       $AppPaths = "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths"
       if(!(Test-Path $AppPaths\$cmd)) {
          $cmd = [IO.Path]::GetFileNameWithoutExtension($cmd)
          if(!(Test-Path $AppPaths\$cmd)){
             $cmd += ".exe"
          }
       }
       if(Test-Path $AppPaths\$cmd) {
          $default = (Get-ItemProperty $AppPaths\$cmd)."(default)"
          if($default) {
             Get-Command $default.Trim("""'")
          }
       }
    }
 #}
`

