---
Author: russellds
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1543
Published Date: 2010-12-17t12
Archived Date: 2015-05-04t21
---

# new-psocustomobject - 

## Description

this function will create a custom psobject using a hashtable as input.

## Comments



## Usage



## TODO



## function

`new-psocustomobject`

## Code

`#
 #
 
 function New-PSOCustomObject {
     param(
           [Parameter(Mandatory=$true)]
           [string]$Name,
           [Parameter(Mandatory=$true)]
           [hashtable]$Properties,
           [Parameter()]
           [string[]]$DefaultProperties,
           [Parameter()]
           [hashtable]$ScriptMethods
          )
     
     $object = New-Object PSObject -Property $Properties
     
     $object.PSObject.TypeNames[0] = "System.Management.Automation.PSCustomObject.$($Name)"
     
     if( $ScriptMethods ) {
         foreach( $key in $ScriptMethods.Keys ) {
             $object | Add-Member -MemberType ScriptMethod -Name $key -Value $ScriptMethods[$key]
         }
     }
     
     if( $DefaultProperties ) {
         Set-PSODefaultProperties -Object $object -DefaultProperties $DefaultProperties
     }
     
     return $object
 }
`

