---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4757
Published Date: 2016-01-02t04
Archived Date: 2016-03-17t04
---

# typealias &amp; accelerators - 

## Description

new-typealias, get-typealias functions for adding and querying type accelerators. compatible with powershell 3.0, 4.0

## Comments



## Usage



## TODO



## function

`new-typealias`

## Code

`#
 #
 
 function New-TypeAlias {
 <#
     .synopsis
         Add a new type alias (accelerator)
     .description
         Add a new type accelerator to the built-in list of accelerators
         like [int], [runspace], [psobject] etc.
     .parameter Type
         The target type to alias (can be an attribute type.)
     .parameter Name
         The alias to use for the target type.
     .inputs
         None. This command does not accept pipeline input.
     .outputs
         None.
 
 #>
     param(
         [parameter(mandatory, position=0)]
         [validatenotnull()]
         [type]$Type,
 
         [parameter(mandatory, position=1)]
         [validatenotnullorempty()]
         [string]$Name
     )
 
     $accel = [psobject].Assembly.GetType("System.Management.Automation.TypeAccelerators")
     $accel::add($Name, $Type)
 
     $accel.getfield("allTypeAccelerators", [reflection.bindingflags]"nonpublic,static").setvalue($accel, $null)
 }
 
 function Get-TypeAlias {
 <#
     .synopsis
         Gets all or a matching subset of type aliases (accelerators)
     .description
         Gets all or a matching subset of user-defined and built-in type
         aliases (accelerators).
 
         You may use wildcards to match the alias name(s).
     .parameter Name
         The alias name to find (optional, may contain wildcards.)
     .inputs
         None. This command does not accept pipeline input.
     .outputs
         Zero or more dictionary entries representing a type alias & type literal pairing.
 #>
     param(
         [parameter(position=0)]
         [validatenotnullorempty()]
         [string]$Name = "*"
     )
     
     $dict = [psobject].Assembly.GetType("System.Management.Automation.TypeAccelerators")::get
     $matches = $dict.Keys | ? { $_ -ilike $Name }
 
     if ($matches) {
         $dict.GetEnumerator() | where key -in $matches
     } elseif (
         -not [System.Management.Automation.WildcardPattern]::ContainsWildcardCharacters($Name))
     {
         write-error -Message "No exact match found for alias '$Name'"
     }
 }
`

