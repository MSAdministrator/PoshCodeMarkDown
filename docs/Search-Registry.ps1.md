---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6886
Published Date: 2017-05-05t07
Archived Date: 2017-05-13t18
---

# search-registry.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

`new-registrymatch`

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
 
 Search the registry for keys or properties that match a specific value.
 
 .EXAMPLE
 
 PS >Set-Location HKCU:\Software\Microsoft\
 PS >Search-Registry Run
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [string] $Pattern
 )
 
 Set-StrictMode -Off
 
 function New-RegistryMatch
 {
     param( $matchType, $keyName, $propertyName, $line )
 
     $registryMatch = New-Object PsObject -Property @{
         MatchType = $matchType;
         KeyName = $keyName;
         PropertyName = $propertyName;
         Line = $line
     }
 
     $registryMatch
 }
 
 foreach($item in Get-ChildItem -Recurse -ErrorAction SilentlyContinue)
 {
     if($item.Name -match $pattern)
     {
         New-RegistryMatch "Key" $item.Name $null $item.Name
     }
 
     foreach($property in (Get-ItemProperty $item.PsPath).PsObject.Properties)
     {
         if(($property.Name -eq "PSPath") -or
             ($property.Name -eq "PSChildName"))
         {
             continue
         }
 
         $propertyText = "$($property.Name)=$($property.Value)"
         if($propertyText -match $pattern)
         {
             New-RegistryMatch "Property" $item.Name `
                 property.Name $propertyText
         }
     }
 }
`

