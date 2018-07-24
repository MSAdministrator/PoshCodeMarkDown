---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 720
Published Date: 2009-12-08t11
Archived Date: 2015-05-06t17
---

# new-type - 

## Description

analogous to one of powershell 2’s add-type cmdlet’s overrides, this script/function will take c# code and compile it in memory.

## Comments



## Usage



## TODO



## script

`new-type`

## Code

`#
 #
 ####################################################################################################
 ####################################################################################################
 ####################################################################################################
    param([string]$TypeDefinition,[string[]]$ReferencedAssemblies)
    
    $provider = New-Object Microsoft.CSharp.CSharpCodeProvider
    $dllName = [PsObject].Assembly.Location
    $compilerParameters = New-Object System.CodeDom.Compiler.CompilerParameters
 
    $assemblies = @("System.dll", $dllName)
    $compilerParameters.ReferencedAssemblies.AddRange($assemblies)
    if($ReferencedAssemblies) { 
       $compilerParameters.ReferencedAssemblies.AddRange($ReferencedAssemblies) 
    }
 
    $compilerParameters.IncludeDebugInformation = $true
    $compilerParameters.GenerateInMemory = $true
 
    $compilerResults = $provider.CompileAssemblyFromSource($compilerParameters, $TypeDefinition)
    if($compilerResults.Errors.Count -gt 0) {
      $compilerResults.Errors | % { Write-Error ("{0}:`t{1}" -f $_.Line,$_.ErrorText) }
    }
 #}
`

