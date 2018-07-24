---
Author: cglessner
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 747
Published Date: 2009-12-22t10
Archived Date: 2012-10-02t04
---

# ps2wcf - 

## Description

call wcf services with powershell using any binding. generates proxy on the fly without needing any tool expect .net 3.5. you can also discover the service endpoints, bindings and contracts. read more on my blog

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.ServiceModel")
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.Runtime.Serialization")
 
 function global:Get-WsdlImporter($wsdlUrl=$(throw "parameter -wsdlUrl is missing"), $httpGet)
 {
 	if($httpGet -eq $true)
 	{
 		$local:mode = [System.ServiceModel.Description.MetadataExchangeClientMode]::HttpGet
 	}
 	else
 	{
 		$local:mode = [System.ServiceModel.Description.MetadataExchangeClientMode]::MetadataExchange
 	}
 	
 	$mexClient = New-Object System.ServiceModel.Description.MetadataExchangeClient((New-Object System.Uri($wsdlUrl)),$mode)
 	$mexClient.MaximumResolvedReferences = [System.Int32]::MaxValue
 	$metadataSet = $mexClient.GetMetadata()
 	$wsdlImporter = New-Object System.ServiceModel.Description.WsdlImporter($metadataSet)
 	
 	return $wsdlImporter	
 }
 
 function global:Get-WcfProxy($wsdlImporter=$null, $wsdlUrl, $proxyPath)
 {
 	if($wsdlImporter -eq $null -and $wsdlUrl -eq $null)
 	{
 		throw "parameter -wsdlImporter or -wsdlUrl must be specified"
 	}
 	else
 	{
 		if($wsdlImporter -eq $null)
 		{
 			$wsdlImporter = Get-WsdlImporter -wsdlUrl $wsdlUrl
 			trap [Exception]
 			{
 				$script:wsdlImporter = Get-WsdlImporter -wsdlUrl $wsdlUrl -httpGet $true
 				continue
 			}
 		}
 	}
 	
 	$generator = new-object System.ServiceModel.Description.ServiceContractGenerator
 	
 	foreach($contractDescription in $wsdlImporter.ImportAllContracts())
 	{
 		[void]$generator.GenerateServiceContractType($contractDescription)
 	}
 	
 	$parameters = New-Object System.CodeDom.Compiler.CompilerParameters
 	if($proxyPath -eq $null)
 	{
 		[void]$parameters.GenerateInMemory = $true
 	}
 	else
 	{
 		$parameters.OutputAssembly = $proxyPath
 	}
 	
 	$providerOptions = new-object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
 	[void]$providerOptions.Add("CompilerVersion","v3.5")
 	
 	$compiler = New-Object Microsoft.CSharp.CSharpCodeProvider($providerOptions)
 	$result = $compiler.CompileAssemblyFromDom($parameters, $generator.TargetCompileUnit);
 	
 	if($result.Errors.Count -gt 0)
 	{
 		throw "Proxy generation failed"       
 	}
 	
 	foreach($type in $result.CompiledAssembly.GetTypes())
 	{
 		if($type.BaseType.IsGenericType)
 		{
 			if($type.BaseType.GetGenericTypeDefinition().FullName -eq "System.ServiceModel.ClientBase``1" )
 			{
 				$type
 			}
 		}
 	}
 }
`

