---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2200
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# new-genericobject.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

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
 
 Creates an object of a generic type:
 
 .EXAMPLE
 
 PS >New-GenericObject System.Collections.ObjectModel.Collection System.Int32
 Creates a simple generic collection
 
 .EXAMPLE
 
 PS >New-GenericObject System.Collections.Generic.Dictionary `
       System.String,System.Int32
 Creates a generic dictionary with two types
 
 .EXAMPLE
 
 PS >$secondType = New-GenericObject System.Collections.Generic.List Int32
 PS >New-GenericObject System.Collections.Generic.Dictionary `
       System.String,$secondType.GetType()
 Creates a generic list as the second type to a generic dictionary
 
 .EXAMPLE
 
 PS >New-GenericObject System.Collections.Generic.LinkedListNode `
       System.String "Hi"
 Creates a generic type with a non-default constructor
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [string] $TypeName,
 
     [Parameter(Mandatory = $true)]
     [string[]] $TypeParameters,
 
     [object[]] $ConstructorParameters
 )
 
 Set-StrictMode -Version Latest
 
 $genericTypeName = $typeName + '`' + $typeParameters.Count
 $genericType = [Type] $genericTypeName
 
 if(-not $genericType)
 {
     throw "Could not find generic type $genericTypeName"
 }
 
 [type[]] $typedParameters = $typeParameters
 $closedType = $genericType.MakeGenericType($typedParameters)
 if(-not $closedType)
 {
     throw "Could not make closed type $genericType"
 }
 
 ,[Activator]::CreateInstance($closedType, $constructorParameters)
`

