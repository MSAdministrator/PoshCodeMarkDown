---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1976
Published Date: 
Archived Date: 2010-07-18t09
---

# set-assemblybindlogging - 

## Description

enable or disable assembly bind logging (per-machine)

## Comments



## Usage



## TODO



## function

`set-assemblybindlogging`

## Code

`#
 #
 function Set-AssemblyBindLogging {
 #.Synopsis
 #.Parameter EnableLogging
 #.Parameter LogPath
 [CmdletBinding()]
    param( 
       [Parameter(Mandatory=$true)]
       #[ValidateSet("Enable","Disable","True","False","0","1")]
       [string]$EnableLogging
    ,
       $LogPath = "C:\Logs\Fusion" 
    )
 
    foreach($RegistryRoot in "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Fusion","HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Fusion") {
       switch -regex ($EnableLogging) {
          "En?a?b?l?e?|Tr?u?e?|1" {
             mkdir $LogPath -Force -EA Stop
 
             Set-ItemProperty REGISTRY::$RegistryRoot LogPath $LogPath
             foreach($switch in "LogFailures","ForceLog","LogResourceBinds") {
                Set-ItemProperty REGISTRY::$RegistryRoot $switch 1 -Type DWord
             }
          }
          "Di?s?a?b?l?e?|Fa?l?s?e?|0" {
             rmdir $LogPath
             foreach($switch in "LogPath","LogFailures","ForceLog","LogResourceBinds") {
                Remove-ItemProperty $switch
             }
          }
          default {
             throw "Couldn't convert '$EnableLogging' to a valid value. Valid values are: Enable or Disable, True or False, 1 or 0."
          }
       }
    }  
 }
`

