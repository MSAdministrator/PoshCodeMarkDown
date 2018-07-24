---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 190
Published Date: 2009-05-02t07
Archived Date: 2016-03-03t14
---

# new-struct - 

## Description

a code-generating and emitting magic function for creating type-safe struct classes for use in powershell!

## Comments



## Usage



## TODO



## class

`new-struct`

## Code

`#
 #
 ##
 function New-Struct {
 	param([string]$Name,[HashTable]$Properties)
 	switch($Properties.Keys){{$_ -isnot [String]}{throw "Invalid Syntax"}}
 	switch($Properties.Values){{$_ -isnot [type]}{throw "Invalid Syntax"}}
 
 	$code = @"
 using System;
 public struct $Name {
 
   $($Properties.Keys | % { "  public {0} {1};`n" -f $Properties[$_],($_.ToUpper()[0] + $_.SubString(1)) })
   public $Name ($( [String]::join(',',($Properties.Keys | % { "{0} {1}" -f $Properties[$_],($_.ToLower()) })) )) {
     $($Properties.Keys | % { "    {0} = {1};`n" -f ($_.ToUpper()[0] + $_.SubString(1)),($_.ToLower()) })
   }
 }
 "@
 
 	$provider = New-Object Microsoft.CSharp.CSharpCodeProvider
 	$dllName = [PsObject].Assembly.Location
 	$compilerParameters = New-Object System.CodeDom.Compiler.CompilerParameters
 	$assemblies = @("System.dll", $dllName)
 
 	$compilerParameters.ReferencedAssemblies.AddRange($assemblies)
 	$compilerParameters.IncludeDebugInformation = $true
 	$compilerParameters.GenerateInMemory = $true
 
 	$compilerResults = $provider.CompileAssemblyFromSource($compilerParameters, $code)
 	if($compilerResults.Errors.Count -gt 0) {
 	  $compilerResults.Errors | % { Write-Error $_.Line + ":`t" + $_.ErrorText }
 	}
 }
`

