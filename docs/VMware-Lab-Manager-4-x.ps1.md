---
Author: oldsienna
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2318
Published Date: 2011-10-24t11
Archived Date: 2015-05-09t14
---

# vmware lab manager 4.x - 

## Description

based on poshcode.org/753 – minor mods to support new mandatory authentication parameters in lab manager 4.x.

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
 	param
     (
         [string] $server, 
         $credential,
         [string] $organizationname = "Default",
         [string] $workspacename = "Main"
     )
         
 	$server = "https://" + $server + "/"
 	$endpoint = $server + "LabManager/SOAP/LabManager.asmx"
 	$proxy = new-webserviceproxy -uri $endpoint -cred $credential
 
 	$authHeader = New-ObjectFromProxy -proxy $proxy -proxyAttributeName "AuthenticationHeaderValue" -typeName "AuthenticationHeader"
 	$authHeader.username = $credential.GetNetworkCredential().UserName
 	$authHeader.password = $credential.GetNetworkCredential().Password
     $authHeader.organizationname = $organizationname
     $authHeader.workspacename = $workspacename
 	$proxy.AuthenticationHeaderValue = $authHeader
 	return $proxy
 }
 
 function Get-LabManagerInternal
 {
 	param
 	(
 		[string] $server = $(throw "Parameter -Server [System.String] is required."),
 		$credential = $(get-credential),
 		[string] $organizationname = "Global",
 		[string] $workspacename = "Main"
 	)
 	
 	$labManagerInternalUri = [System.Uri] "https://$server/LabManager/SOAP/LabManagerInternal.asmx"
 	$proxy = New-WebServiceProxy -Uri $labManagerInternalUri -Credential $credential
 	
 	if ($proxy)
 	{
 		$authHeader = New-ObjectFromProxy -proxy $proxy -proxyAttributeName "AuthenticationHeaderValue" -typeName "AuthenticationHeader"
 		$authHeader.username = $credential.GetNetworkCredential().UserName
 		$authHeader.password = $credential.GetNetworkCredential().Password
 		$authHeader.organizationname = $organizationname
 		$authHeader.workspacename = $workspacename
 		$proxy.AuthenticationHeaderValue = $authHeader
 		return $proxy
 	}
 }
 
 Ignore-SslErrors
 
 $labManager = Connect-LabManager -server "demo.eng.vmware.com" -credential (get-credential)
 
 $labManager | gm | where { $_.MemberType -eq "Method" }
 
 $labManager.ListConfigurations(1)
 
 $labManager.ListConfigurations(1) | foreach { write-host ("For Configuration " + $_.id + ":"); $labManager.ListMachines($_.id) }
 
 $labmanagerinternal = Get-LabManagerInternal -server "demo.eng.vmware.com" -organizationname "Default" -workspacename "Main" -credential (get-credential)
 	
 $labmanagerinternal.GetAllMedia()
`

