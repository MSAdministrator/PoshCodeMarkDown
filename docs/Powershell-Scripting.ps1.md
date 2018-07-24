---
Author: praveen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5789
Published Date: 2016-03-17t10
Archived Date: 2016-03-06t00
---

# powershell scripting - 

## Description

can anyone help me here and explain the below code

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function filecount {            
 param (            
  [string]$path            
 )            
  if (-not (Test-Path $path)){Throw "Path: $path not found"}            
              
  $count = 0            
  $count = Get-ChildItem -Path $path |             
           where {!$_.PSIsContainer} |             
           Measure-Object |            
           select -Expand count            
                       
  Get-Item -Path $path |           
  select PSDrive,             
  ##@{N="Parent"; E={($_.PSParentPath -split "FileSystem::")[1]}},            
  Name,            
  @{N="FileCount"; E={$count}} | Where-Object { !($_ -match 'psa4ruww') }
  Get-ChildItem -Path $path | 
             
  where {$_.PSIsContainer} |             
  foreach {
    filecount $($_.Fullname)            
  }            
             
 }             
             
 filecount "dir path"
`

