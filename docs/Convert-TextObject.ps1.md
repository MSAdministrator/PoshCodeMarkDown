---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2134
Published Date: 2011-09-09t21
Archived Date: 2017-04-30t11
---

# convert-textobject.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

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
 
 Convert a simple string into a custom PowerShell object.
 
 .EXAMPLE
 
 "Hello World" | Convert-TextObject
 Generates an Object with "P1=Hello" and "P2=World"
 
 .EXAMPLE
 
 "Hello World" | Convert-TextObject -Delimiter "ll"
 Generates an Object with "P1=He" and "P2=o World"
 
 .EXAMPLE
 
 "Hello World" | Convert-TextObject -Pattern "He(ll.*o)r(ld)"
 Generates an Object with "P1=llo Wo" and "P2=ld"
 
 .EXAMPLE
 
 "Hello World" | Convert-TextObject -PropertyName FirstWord,SecondWord
 Generates an Object with "FirstWord=Hello" and "SecondWord=World
 
 .EXAMPLE
 
 "123 456" | Convert-TextObject -PropertyType $([string],[int])
 Generates an Object with "Property1=123" and "Property2=456"
 The second property is an integer, as opposed to a string
 
 .EXAMPLE
 
 PS >$ipAddress = (ipconfig | Convert-TextObject -Delim ": ")[2].P2
 PS >$ipAddress
 192.168.1.104
 
 #>
 
 [CmdletBinding(DefaultParameterSetName = "ByDelimiter")]
 param(
     [Parameter(ParameterSetName = "ByDelimiter", Position = 0)]
     [string] $Delimiter = "\s+",
 
     [Parameter(Mandatory = $true,
         ParameterSetName = "ByPattern",
         Position = 0)]
     [string] $Pattern,
 
     [Parameter(Position = 1)]
     [Alias("PN")]
     [string[]] $PropertyName = @(),
 
     [Parameter(Position = 2)]
     [Alias("PT")]
     [type[]] $PropertyType = @(),
 
     [Parameter(ValueFromPipeline = $true)]
     [string] $InputObject
 )
 
 begin {
     Set-StrictMode -Version Latest
 }
 
 process {
     $returnObject = New-Object PSObject
 
     $matches = $null
     $matchCount = 0
 
     if($PSBoundParameters["Pattern"])
     {
         if(-not ($InputObject -match $pattern))
         {
             return
         }
 
         $matchCount = $matches.Count
     $startIndex = 1
     }
     else
     {
         if(-not ($InputObject -match $delimiter))
         {
             return
         }
 
         $matches = $InputObject -split $delimiter
         $matchCount = $matches.Length
         $startIndex = 0
     }
 
     for($counter = $startIndex; $counter -lt $matchCount; $counter++)
     {
         $currentPropertyName = "P$($counter - $startIndex + 1)"
         $currentPropertyType = [string]
 
         if($counter -lt $propertyName.Length)
         {
             if($propertyName[$counter])
             {
                 $currentPropertyName = $propertyName[$counter - 1]
             }
         }
 
         if($counter -lt $propertyType.Length)
         {
             if($propertyType[$counter])
             {
                 $currentPropertyType = $propertyType[$counter - 1]
             }
         }
 
         Add-Member -InputObject $returnObject NoteProperty `
             -Name $currentPropertyName `
             -Value ($matches[$counter].Trim() -as $currentPropertyType)
     }
 
     $returnObject
 }
`

