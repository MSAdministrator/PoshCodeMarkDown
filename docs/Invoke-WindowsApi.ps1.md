---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2189
Published Date: 2011-09-09t21
Archived Date: 2017-03-10t08
---

# invoke-windowsapi.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Invoke a native Windows API call that takes and returns simple data types.
 
 
 .EXAMPLE
 
 PS >$filename = "c:\temp\hardlinked.txt"
 PS >$existingFilename = "c:\temp\link_target.txt"
 PS >Set-Content $existingFilename "Hard Link target"
 PS >$parameterTypes = [string], [string], [IntPtr]
 PS >$parameters = [string] $filename, [string] $existingFilename,
     [IntPtr]::Zero
 
 PS >$result = Invoke-WindowsApi "kernel32" ([bool]) "CreateHardLink" `
     $parameterTypes $parameters
 PS >Get-Content C:\temp\hardlinked.txt
 Hard Link target
 
 #>
 
 param(
     [string] $DllName,
 
     [Type] $ReturnType,
 
     [string] $MethodName,
 
     [Type[]] $ParameterTypes,
 
     [Object[]] $Parameters
 )
 
 Set-StrictMode -Version Latest
 
 $domain = [AppDomain]::CurrentDomain
 $name = New-Object Reflection.AssemblyName 'PInvokeAssembly'
 $assembly = $domain.DefineDynamicAssembly($name, 'Run')
 $module = $assembly.DefineDynamicModule('PInvokeModule')
 $type = $module.DefineType('PInvokeType', "Public,BeforeFieldInit")
 
 $inputParameters = @()
 $refParameters = @()
 
 for($counter = 1; $counter -le $parameterTypes.Length; $counter++)
 {
     if($parameterTypes[$counter - 1] -eq [Ref])
     {
         $refParameters += $counter
 
         $parameterTypes[$counter - 1] =
             $parameters[$counter - 1].Value.GetType().MakeByRefType()
         $inputParameters += $parameters[$counter - 1].Value
     }
     else
     {
         $inputParameters += $parameters[$counter - 1]
     }
 }
 
 $method = $type.DefineMethod(
     $methodName, 'Public,HideBySig,Static,PinvokeImpl',
     $returnType, $parameterTypes)
 foreach($refParameter in $refParameters)
 {
     [void] $method.DefineParameter($refParameter, "Out", $null)
 }
 
 $ctor = [Runtime.InteropServices.DllImportAttribute].GetConstructor([string])
 $attr = New-Object Reflection.Emit.CustomAttributeBuilder $ctor, $dllName
 $method.SetCustomAttribute($attr)
 
 $realType = $type.CreateType()
 
 $realType.InvokeMember(
     $methodName, 'Public,Static,InvokeMethod', $null, $null,$inputParameters)
 
 foreach($refParameter in $refParameters)
 {
     $parameters[$refParameter - 1].Value = $inputParameters[$refParameter - 1]
 }
`

