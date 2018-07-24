---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2991
Published Date: 2012-10-06t21
Archived Date: 2015-05-29t08
---

# protect-variable - 

## Description

a function to make it easier (in powershell 2) to enforce rules about variables (without making them parameters).

## Comments



## Usage



## TODO



## function

`protect-variable`

## Code

`#
 #
 function Protect-Variable {
 param(
 [Parameter(Mandatory=$true,Position=0)][String]$Name, [Int]$Scope = 1,
 [Parameter(ParameterSetName="ValidateScriptBlock")][ScriptBlock]$ScriptBlock,
 [Parameter(ParameterSetName="ValidateCount")][Int]$MinCount,
 [Parameter(ParameterSetName="ValidateCount")][Int]$MaxCount,
 [Parameter(ParameterSetName="ValidateLength")][Int]$MinLength,
 [Parameter(ParameterSetName="ValidateLength")][Int]$MaxLength,
 [Parameter(ParameterSetName="ValidatePattern")][Regex]$Pattern,
 [Parameter(ParameterSetName="ValidateRange")][Int]$MinValue,
 [Parameter(ParameterSetName="ValidateRange")][Int]$MaxValue,
 [Parameter(ParameterSetName="ValidateSet")][String[]]$Set,
 [Parameter(ParameterSetName="ValidateNotNull")][Switch]$NotNull,
 [Parameter(ParameterSetName="ValidateNotNullOrEmpty")][Switch]$NotEmpty
 )
    $Variable = Get-Variable $Name -Scope 1
    
    if($ScriptBlock) {
       $Attribute = new-object System.Management.Automation.ValidateScriptAttribute $ScriptBlock
       $Variable.Attributes.Add($Attribute)
    }
    if($MinCount -or $MaxCount) {
       if(!$MinCount) { $MinCount = [Int]::MinValue }
       if(!$MaxCount) { $MaxCount = [Int]::MaxValue }
       $Attribute = new-object System.Management.Automation.ValidateCountAttribute $MinCount, $MaxCount
       $Variable.Attributes.Add($Attribute)
    }
    if($MinLength -or $MaxLength) {
       $Attribute = new-object System.Management.Automation.ValidateLengthAttribute $MinLength, $MaxLength
       $Variable.Attributes.Add($Attribute)
    }
    if($Pattern) {
       $Attribute = new-object System.Management.Automation.ValidatePatternAttribute $Pattern
       $Variable.Attributes.Add($Attribute)
    }
    if($MinValue -or $MaxValue) {
       $Attribute = new-object System.Management.Automation.ValidateRangeAttribute $MinValue, $MaxValue
       $Variable.Attributes.Add($Attribute)
    }
    if($Set) {
       $Attribute = new-object System.Management.Automation.ValidateSetAttribute $Set
       $Variable.Attributes.Add($Attribute)
    }
    if($NotNull) {
       $Attribute = new-object System.Management.Automation.ValidateNotNullAttribute
       $Variable.Attributes.Add($Attribute)
    }
    if($NotEmpty) {
       $Attribute = new-object System.Management.Automation.ValidateNotNullOrEmptyAttribute
       $Variable.Attributes.Add($Attribute)
    }
 }
`

