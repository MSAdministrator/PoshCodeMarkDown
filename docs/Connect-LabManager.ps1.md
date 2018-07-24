---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 753
Published Date: 2009-12-27t21
Archived Date: 2016-07-11t02
---

# connect-labmanager - 

## Description

if you have vmware lab manager, this script makes it easier than ever to connect to and automate lab manager actions.

## Comments



## Usage



## TODO



## function

`ignore-sslerrors`

## Code

`#
 #
 function Ignore-SslErrors {
 	$Provider=New-Object Microsoft.CSharp.CSharpCodeProvider
 	$Compiler=$Provider.CreateCompiler()
 	$Params=New-Object System.CodeDom.Compiler.CompilerParameters
 	$Params.GenerateExecutable=$False
 	$Params.GenerateInMemory=$True
 	$Params.IncludeDebugInformation=$False
 	$Params.ReferencedAssemblies.Add("System.DLL") > $null
 	$TASource=@'
 	  namespace Local.ToolkitExtensions.Net.CertificatePolicy {
 	    public class TrustAll : System.Net.ICertificatePolicy {
 	      public TrustAll() { 
 	      }
 	      public bool CheckValidationResult(System.Net.ServicePoint sp,
 	        System.Security.Cryptography.X509Certificates.X509Certificate cert, 
 	        System.Net.WebRequest req, int problem) {
 	        return true;
 	      }
 	    }
 	  }
 '@ 
 	$TAResults=$Provider.CompileAssemblyFromSource($Params,$TASource)
 	$TAAssembly=$TAResults.CompiledAssembly
 
 	$TrustAll=$TAAssembly.CreateInstance("Local.ToolkitExtensions.Net.CertificatePolicy.TrustAll")
 	[System.Net.ServicePointManager]::CertificatePolicy=$TrustAll
 }
 
 function New-ObjectFromProxy {
 	param($proxy, $proxyAttributeName, $typeName)
 
 	$attribute = $proxy | gm | where { $_.Name -eq $proxyAttributeName }
 	$str = "`$assembly = [" + $attribute.TypeName + "].assembly"
 	invoke-expression $str
 
 	$type = $assembly.getTypes() | where { $_.Name -eq $typeName }
 	return $assembly.CreateInstance($type)
 }
 
 function Connect-LabManager {
 	param($server, $credential)
 
 	$server = "https://" + $server + "/"
 	$endpoint = $server + "LabManager/SOAP/LabManager.asmx"
 	$proxy = new-webserviceproxy -uri $endpoint -cred $credential
 
 	$authHeader = New-ObjectFromProxy -proxy $proxy -proxyAttributeName "AuthenticationHeaderValue" -typeName "AuthenticationHeader"
 	$authHeader.username = $credential.GetNetworkCredential().UserName
 	$authHeader.password = $credential.GetNetworkCredential().Password
 	$proxy.AuthenticationHeaderValue = $authHeader
 	return $proxy
 }
 
 
 
 
 
`

