---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 68
Published Date: 2008-12-09t11
Archived Date: 2016-03-10t08
---

# invoke-inline - 

## Description

this is lee holmes’ invoke-inline, with some minor modifications by joel bennett to allow it to use #include statements, and to return collections.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##
 param(
     [string[]] $code, 
     [object[]] $arguments,
     [string[]] $reference = @()
     )
 
 if(-not (Test-Path Variable:\inlineCode.Cache))
 {
     ${GLOBAL:inlineCode.Cache} = @{}
 }
 
 $using = $null;
 $source = $null;
 if($code.length -eq 1) {
 	$source = $code[0]
 } elseif($code.Length -eq 2){
 	$using = $code[0]
 	$source = $code[1]
 } else {
 	Write-Error "You have to pass some code, or this won't do anything ..."
 }
 
 $params = @()
 foreach($arg in $arguments) { $params += $arg }
 $arguments = $params
 
 
 function main
 {
     $cachedObject = ${inlineCode.Cache}[$source] 
 	Write-Verbose "Type: $($arguments[0].GetType())"
 
     if($cachedObject -eq $null)
     {
         $codeToCompile = 
 @"
     using System;
 	using System.Collections.Generic;
 	$using
 
     public class InlineRunner 
     { 
         public List<object> Invoke(Object[] args) 
         { 
             List<object> result = new List<object>(); 
 
             $source 
 
             if( result.Count > 0 ) {
 				return result;
 			} else {
 				return null;
 			}
         } 
     } 
 "@
 		Write-Verbose $codeToCompile
 
         $provider = New-Object Microsoft.CSharp.CSharpCodeProvider 
 
         $dllName = [PsObject].Assembly.Location
 
         $compilerParameters = New-Object System.CodeDom.Compiler.CompilerParameters 
 
         $assemblies = @("System.dll", $dllName) 
         $compilerParameters.ReferencedAssemblies.AddRange($assemblies) 
         $compilerParameters.ReferencedAssemblies.AddRange($reference)
         $compilerParameters.IncludeDebugInformation = $true 
         $compilerParameters.GenerateInMemory = $true 
 
         $compilerResults =
             $provider.CompileAssemblyFromSource($compilerParameters, $codeToCompile) 
 
         if($compilerResults.Errors.Count -gt 0) 
         { 
             $errorLines = "" 
             foreach($error in $compilerResults.Errors) 
             { 
                 $errorLines += "`n`t" + $error.Line + ":`t" + $error.ErrorText 
             } 
             Write-Error $errorLines 
         } 
         else 
         { 
             ${inlineCode.Cache}[$source] = 
                 $compilerResults.CompiledAssembly.CreateInstance("InlineRunner") 
         } 
 
         $cachedObject = ${inlineCode.Cache}[$source] 
    } 
 
    Write-Verbose "Argument $arguments`n`n$cachedObject"
    if($cachedObject -ne $null)
    { 
        return $cachedObject.Invoke($arguments)
    } 
 }
 . Main
`

