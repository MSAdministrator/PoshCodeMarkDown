---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1862
Published Date: 
Archived Date: 2010-05-24t16
---

# get-constructor - 

## Description

enumerates the constructors of a type with the parameters that they take, so you can figure out what your options are when calling new-object. now outputs fully paste-able syntax ;)

## Comments



## Usage



## TODO



## function

`get-constructor`

## Code

`#
 #
 function Get-Constructor {
 PARAM( [Type]$type, [Switch]$Simple)
    if($Simple) {
    $type.GetConstructors() | 
       Select @{
          l="$($type.Name) Constructor"
          e={ "New-Object $($type.FullName) $(($_.GetParameters() | ForEach { "[{0}]`${1}" -f $_.ParameterType.FullName, $_.Name }) -Join ", ")" }
       }
    } else {
    $type.GetConstructors() | 
       Select @{
          name = "$($type.Name) Constructors"
          expression = { 
             $parameters = $(
                foreach($param in $_.GetParameters()) {
                   Write-Host $param.ParameterType.FullName.TrimEnd('&'), $param.Name -fore cyan
                   
                   if($param.ParameterType.Name.EndsWith('&')) { $ref = '[ref]' } else { $ref = '' }
                   if($param.ParameterType.IsArray) { $array = ',' } else { $array = '' }
                   '{0}({1}[{2}]${3})' -f $ref, $array, $param.ParameterType.FullName.TrimEnd('&'), $param.Name
                }
             )
          
             "New-Object $($type.FullName) $($parameters -join ', ')"
          }
       }
    }
 }
 
 Set-Alias gctor Get-Constructor
`

