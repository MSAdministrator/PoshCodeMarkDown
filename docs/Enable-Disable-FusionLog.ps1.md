---
Author: inwza678
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6367
Published Date: 2016-06-03t05
Archived Date: 2016-06-04t19
---

# enable/disable fusionlog - 

## Description

http

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function global:Enable-FusionLog {  
     Remove-ItemProperty HKLM:Software\Microsoft\Fusion -name EnableLog -ErrorAction SilentlyContinue  
     [void](New-ItemProperty  HKLM:Software\Microsoft\Fusion -name EnableLog -propertyType dword -ErrorAction Stop)  
     Set-ItemProperty  HKLM:Software\Microsoft\Fusion -name EnableLog -value 1 -ErrorAction Stop  
     Write-Host "Fusion log enabled.  Do not forget to disable it when you are done"  
 }  
 function global:Disable-FusionLog {  
     Remove-ItemProperty HKLM:Software\Microsoft\Fusion -name EnableLog -ErrorAction SilentlyContinue  
     Write-Host "Fusion log disabled"  
 }
`

