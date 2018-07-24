---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2169
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# get-wmiclasskeyproperty. - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Get all of the properties that you may use in a WMI filter for a given class.
 
 .EXAMPLE
 
 Get-WmiClassKeyProperty Win32_Process
 Handle
 
 #>
 
 param(
     [WmiClass] $WmiClass
 )
 
 Set-StrictMode -Version Latest
 
 foreach($currentProperty in $wmiClass.Properties)
 {
     foreach($qualifier in $currentProperty.Qualifiers)
     {
         if($qualifier.Name -eq "Key")
         {
             $currentProperty.Name
         }
     }
 }
`

