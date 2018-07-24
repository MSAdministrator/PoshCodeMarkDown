---
Author: matthew graeber
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3625
Published Date: 2013-09-06t07
Archived Date: 2016-07-13t12
---

# disable-sslvalidation - 

## Description

disable-sslvalidation is built as a workaround for invoke-webrequest errors when attempting to get content from a site using a self-signed ssl certificate.

## Comments



## Usage



## TODO



## function

`disable-sslvalidation`

## Code

`#
 #
 function Disable-SSLValidation
 {
 <#
 .SYNOPSIS
     Disables SSL certificate validation
 .DESCRIPTION
     Disable-SSLValidation disables SSL certificate validation by using reflection to implement the System.Net.ICertificatePolicy class.
 
     Author: Matthew Graeber (@mattifestation)
     License: BSD 3-Clause
 .NOTES
 .LINK
     http://www.exploit-monday.com
 #>
 
     Set-StrictMode -Version 2
 
     if ([System.Net.ServicePointManager]::CertificatePolicy.ToString() -eq 'IgnoreCerts') { Return }
 
     $Domain = [AppDomain]::CurrentDomain
     $DynAssembly = New-Object System.Reflection.AssemblyName('IgnoreCerts')
     $AssemblyBuilder = $Domain.DefineDynamicAssembly($DynAssembly, [System.Reflection.Emit.AssemblyBuilderAccess]::Run)
     $ModuleBuilder = $AssemblyBuilder.DefineDynamicModule('IgnoreCerts', $false)
     $TypeBuilder = $ModuleBuilder.DefineType('IgnoreCerts', 'AutoLayout, AnsiClass, Class, Public, BeforeFieldInit', [System.Object], [System.Net.ICertificatePolicy])
     $TypeBuilder.DefineDefaultConstructor('PrivateScope, Public, HideBySig, SpecialName, RTSpecialName') | Out-Null
     $MethodInfo = [System.Net.ICertificatePolicy].GetMethod('CheckValidationResult')
     $MethodBuilder = $TypeBuilder.DefineMethod($MethodInfo.Name, 'PrivateScope, Public, Virtual, HideBySig, VtableLayoutMask', $MethodInfo.CallingConvention, $MethodInfo.ReturnType, ([Type[]] ($MethodInfo.GetParameters() | % {$_.ParameterType})))
     $ILGen = $MethodBuilder.GetILGenerator()
     $ILGen.Emit([Reflection.Emit.Opcodes]::Ldc_I4_1)
     $ILGen.Emit([Reflection.Emit.Opcodes]::Ret)
     $TypeBuilder.CreateType() | Out-Null
 
     [System.Net.ServicePointManager]::CertificatePolicy = New-Object IgnoreCerts
 }
`

