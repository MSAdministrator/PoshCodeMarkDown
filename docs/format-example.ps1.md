---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1780
Published Date: 
Archived Date: 2010-04-17t07
---

# format example - 

## Description

quick code example for a discussion on posting code.  code lifted from http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $packages = dir "C:\Program Files\Microsoft SQL Server\100\DTS\Packages\*" | select -ExpandProperty Fullname | foreach {get-ispackage -path $_ }
 $packages | foreach {$package = $_; $_.Configurations | Select @{n='Package';e={$Package.DisplayName}}, Name,ConfigurationString}
 $packages | foreach {$package = $_; $_.Connections | Select @{n='Package';e={$Package.DisplayName}}, Name,ConnectionString}
 
 $packages = Get-ChildItem -Path "C:\Program Files\Microsoft SQL Server\100\DTS\Packages\*" | 
     Select-Object -ExpandProperty Fullname | 
         ForEach-Object {
             get-ispackage -path $_ 
         }
 foreach ($package in $packages)
 {
     $package.Configurations | 
         Select-Object -Property Name, ConfigurationString, @{
             Name='Package'
             Expression={$Package.DisplayName}
         }
 }
 foreach ($package in $packages)
 {
     $package.Connections | 
         Select-Object -Property Name, ConfigurationString, @{
             Name='Package'
             Expression={$Package.DisplayName}
         }
 }
`

