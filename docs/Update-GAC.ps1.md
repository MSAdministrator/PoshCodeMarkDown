---
Author: jeffrey snover
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 577
Published Date: 
Archived Date: 2008-09-19t16
---

# update-gac - 

## Description

latest version from http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Set-Alias ngen (Join-Path ([System.Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory()) ngen.exe)
 [AppDomain]::CurrentDomain.GetAssemblies() |
     sort {Split-path $_.location -leaf} | 
     %{
         $Name = (Split-Path $_.location -leaf)
         if ([System.Runtime.InteropServices.RuntimeEnvironment]::FromGlobalAccessCache($_))
         {
             Write-Host "Already GACed: $Name"
         }else
         {
             Write-Host -ForegroundColor Yellow "NGENing      : $Name"
             ngen $_.location | %{"`t$_"}
          }
       }
`

