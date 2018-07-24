---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2133
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t07
---

# connect-webservice.ps1 - 

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
 ##
 ##
 ##
 ##############################################################################
 
 param(
     [string] $WsdlLocation = $(throw "Please specify a WSDL location"),
 
     [string] $Namespace,
 
     [Switch] $RequiresAuthentication
 )
 
 if(-not (Test-Path Variable:\Lee.Holmes.WebServiceCache))
 {
     ${GLOBAL:Lee.Holmes.WebServiceCache} = @{}
 }
 
 $oldInstance = ${GLOBAL:Lee.Holmes.WebServiceCache}[$wsdlLocation]
 if($oldInstance)
 {
     $oldInstance
     return
 }
 
 Add-Type -Assembly System.Web.Services
 
 $wc = New-Object System.Net.WebClient
 
 if($requiresAuthentication)
 {
     $wc.UseDefaultCredentials = $true
 }
 
 $wsdlStream = $wc.OpenRead($wsdlLocation)
 
 if(-not (Test-Path Variable:\wsdlStream))
 {
     return
 }
 
 $serviceDescription =
     [Web.Services.Description.ServiceDescription]::Read($wsdlStream)
 $wsdlStream.Close()
 
 if(-not (Test-Path Variable:\serviceDescription))
 {
     return
 }
 
 $serviceNamespace = New-Object System.CodeDom.CodeNamespace
 if($namespace)
 {
     $serviceNamespace.Name = $namespace
 }
 
 $codeCompileUnit = New-Object System.CodeDom.CodeCompileUnit
 $serviceDescriptionImporter =
     New-Object Web.Services.Description.ServiceDescriptionImporter
 $serviceDescriptionImporter.AddServiceDescription(
     $serviceDescription, $null, $null)
 [void] $codeCompileUnit.Namespaces.Add($serviceNamespace)
 [void] $serviceDescriptionImporter.Import(
     $serviceNamespace, $codeCompileUnit)
 
 $generatedCode = New-Object Text.StringBuilder
 $stringWriter = New-Object IO.StringWriter $generatedCode
 $provider = New-Object Microsoft.CSharp.CSharpCodeProvider
 $provider.GenerateCodeFromCompileUnit($codeCompileUnit, $stringWriter, $null)
 
 $references = @("System.dll", "System.Web.Services.dll", "System.Xml.dll")
 $compilerParameters = New-Object System.CodeDom.Compiler.CompilerParameters
 $compilerParameters.ReferencedAssemblies.AddRange($references)
 $compilerParameters.GenerateInMemory = $true
 
 $compilerResults =
     $provider.CompileAssemblyFromSource($compilerParameters, $generatedCode)
 
 if($compilerResults.Errors.Count -gt 0)
 {
     $errorLines = ""
     foreach($error in $compilerResults.Errors)
     {
         $errorLines += "`n`t" + $error.Line + ":`t" + $error.ErrorText
     }
 
     Write-Error $errorLines
     return
 }
 else
 {
     $assembly = $compilerResults.CompiledAssembly
 
     $type = $assembly.GetTypes() |
         Where-Object { $_.GetCustomAttributes(
             [System.Web.Services.WebServiceBindingAttribute], $false) }
 
     if(-not $type)
     {
         Write-Error "Could not generate web service proxy."
         return
     }
 
     $instance = $assembly.CreateInstance($type)
 
     if($requiresAuthentication)
     {
         if(@($instance.PsObject.Properties |
             where { $_.Name -eq "UseDefaultCredentials" }).Count -eq 1)
         {
             $instance.UseDefaultCredentials = $true
         }
     }
 
     ${GLOBAL:Lee.Holmes.WebServiceCache}[$wsdlLocation] = $instance
 
     $instance
 }
`

