---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1017
Published Date: 
Archived Date: 2009-04-16t01
---

# set-attachedproperty - 

## Description

a piece that’s missing from poshcode 0.01

## Comments



## Usage



## TODO



## function

`set-attachedproperty`

## Code

`#
 #
 [CmdletBinding()]
 PARAM(
    [Parameter(Position=0,Mandatory=$true)
    [System.Windows.DependencyProperty]
    $Property
 ,
    [Parameter(Mandatory=$true,ValueFromPipeline=$true)
    $Element
 )
 DYNAMICPARAM {
    $paramDictionary = new-object System.Management.Automation.RuntimeDefinedParameterDictionary
    $Param1 = new-object System.Management.Automation.RuntimeDefinedParameter
    $Param1.Name = "Value"
    $Param1.Attributes.Add( (New-Object System.Management.Automation.ParameterAttribute -Property @{ Position = 1 }) )
    $Param1.ParameterType = $Property.PropertyType
            
    $paramDictionary.Add("Value", $Param1)
    
    return $paramDictionary
 }
 PROCESS {
    $Element.SetValue($Property, $Param1.Value)
    $Element
 }
 #}
`

