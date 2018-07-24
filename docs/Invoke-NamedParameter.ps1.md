---
Author: jason archer
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2608
Published Date: 2013-04-06t22
Archived Date: 2013-03-31t13
---

# invoke-namedparameter - 

## Description

a function that simplifies calling methods with named parameters to make it easier to deal with long signatures and optional parameters.  this is particularly helpful for com objects.

## Comments



## Usage



## TODO



## function

`invoke-namedparameter`

## Code

`#
 #
 Function Invoke-NamedParameter {
 <#
 .SYNOPSIS
     Invokes a method using named parameters.
 .DESCRIPTION
     A function that simplifies calling methods with named parameters to make it easier to deal with long signatures and optional parameters.  This is particularly helpful for COM objects.
 .PARAMETER Object
     The object that has the method.
 .PARAMETER Method
     The name of the method.
 .PARAMETER Parameter
     A hashtable with the names and values of parameters to use in the method call.
 .PARAMETER Argument
     An array of arguments without names to use in the method call.
 .NOTES
     Name: Invoke-NamedParameter
     Author: Jason Archer
     DateCreated: 2011-04-06
 
     1.0 - 2011-04-06 - Jason Archer
         Initial release.
 .EXAMPLE
     $shell = New-Object -ComObject Shell.Application
     Invoke-NamedParameter $Shell "Explore" @{"vDir"="$pwd"}
 
     Description
     -----------    
     Invokes a method named "Explore" with the named parameter "vDir."
 #>
     [CmdletBinding(DefaultParameterSetName = "Named")]
     param(
         [Parameter(ParameterSetName = "Named", Position = 0, Mandatory = $true)]
         [Parameter(ParameterSetName = "Positional", Position = 0, Mandatory = $true)]
         [ValidateNotNull()]
         [System.Object]$Object
         ,
         [Parameter(ParameterSetName = "Named", Position = 1, Mandatory = $true)]
         [Parameter(ParameterSetName = "Positional", Position = 1, Mandatory = $true)]
         [ValidateNotNullOrEmpty()]
         [String]$Method
         ,
         [Parameter(ParameterSetName = "Named", Position = 2, Mandatory = $true)]
         [ValidateNotNull()]
         [Hashtable]$Parameter
         ,
         [Parameter(ParameterSetName = "Positional")]
         [Object[]]$Argument
     )
 
         if ($PSCmdlet.ParameterSetName -eq "Named") {
             $Object.GetType().InvokeMember($Method, [System.Reflection.BindingFlags]::InvokeMethod,
             )
         } else {
             $Object.GetType().InvokeMember($Method, [System.Reflection.BindingFlags]::InvokeMethod,
             )
         }
     }
 }
 
 <#
 Examples
 
 Calling a method with named parameters.
 
 $shell = New-Object -ComObject Shell.Application
 Invoke-NamedParameters $Shell "Explore" @{"vDir"="$pwd"}
 
 The syntax for more than one would be @{"First"="foo";"Second"="bar"}
 
 Calling a method that takes no parameters (you can also use -Argument with $null).
 
 $shell = New-Object -ComObject Shell.Application
 Invoke-NamedParameters $Shell "MinimizeAll" @{}
 #>
`

