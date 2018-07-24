---
Author: brian
Publisher: 
Copyright: 
Email: 
Version: 4.1.57
Encoding: ascii
License: cc0
PoshCode ID: 6802
Published Date: 2017-03-17t04
Archived Date: 2017-03-23t03
---

# get-websitecertificate - 

## Description

this will return the specified website’s x.509 certificate either as an object or a file (if the -outputfile parameter is specified). see included comment based help for documentation.

## Comments



## Usage



## TODO



## script

`get-websitecertificate`

## Code

`#
 #
 	.NOTES
 	===========================================================================
 	 Created with: 	SAPIEN Technologies, Inc., PowerShell Studio 2014 v4.1.57
 	 Created on:   	11/3/2014 6:45 AM
 	 Created by:   	Brian
 	 Organization: 	
 	 Filename:     	
 	===========================================================================
 	.DESCRIPTION
 		A description of the file.
 #>
 $vcacapfqdn = $args[0]
 
 function Get-WebsiteCertificate {
 	[CmdletBinding()]
 	param (
 		[Parameter(Mandatory = $true)] [System.Uri]
 		$Uri,
 		[Parameter()] [System.IO.FileInfo]
 		$OutputFile,
 		[Parameter()] [Switch]
 		$UseSystemProxy,
 		[Parameter()] [Switch]
 		$UseDefaultCredentials,
 		[Parameter()] [Switch]
 		$TrustAllCertificates
 	)
 	try
 	{
 		$request = [System.Net.WebRequest]::Create($Uri)
 		if ($UseSystemProxy)
 		{
 			$request.Proxy = [System.Net.WebRequest]::DefaultWebProxy
 		}
 		
 		if ($UseSystemProxy -and $UseDefaultCredentials)
 		{
 			$request.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
 		}
 		
 		if ($TrustAllCertificates)
 		{
 			$Provider = New-Object Microsoft.CSharp.CSharpCodeProvider
 			$Compiler = $Provider.CreateCompiler()
 			$Params = New-Object System.CodeDom.Compiler.CompilerParameters
 			$Params.GenerateExecutable = $False
 			$Params.GenerateInMemory = $True
 			$Params.IncludeDebugInformation = $False
 			$Params.ReferencedAssemblies.Add("System.DLL") > $null
 			$TASource = @'
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
 			$TAResults = $Provider.CompileAssemblyFromSource($Params, $TASource)
 			$TAAssembly = $TAResults.CompiledAssembly
 			
 			$TrustAll = $TAAssembly.CreateInstance("Local.ToolkitExtensions.Net.CertificatePolicy.TrustAll")
 			[System.Net.ServicePointManager]::CertificatePolicy = $TrustAll
 		}
 		
 		$response = $request.GetResponse()
 		$servicePoint = $request.ServicePoint
 		$certificate = $servicePoint.Certificate
 		
 		if ($OutputFile)
 		{
 			$certBytes = $certificate.Export(
 			[System.Security.Cryptography.X509Certificates.X509ContentType]::Cert
 			)
 			[System.IO.File]::WriteAllBytes($OutputFile, $certBytes)
 			$OutputFile.Refresh()
 			return $OutputFile
 		}
 		else
 		{
 			return $certificate
 		}
 	}
 	catch
 	{
 		Write-Error "Failed to get website certificate. The error was '$_'."
 		return $null
 	}
 	
 	<#
 		.SYNOPSIS
 			Retrieves the certificate used by a website.
 	
 		.DESCRIPTION
 			Retrieves the certificate used by a website. Returns either an object or file.
 	
 		.PARAMETER  Uri
 			The URL of the website. This should start with https.
 	
 		.PARAMETER  OutputFile
 			Specifies what file to save the certificate as.
 			
 		.PARAMETER  UseSystemProxy
 			Whether or not to use the system proxy settings.
 			
 		.PARAMETER  UseDefaultCredentials
 			Whether or not to use the system logon credentials for the proxy.
 			
 		.PARAMETER  TrustAllCertificates
 			Ignore certificate errors for certificates that are expired, have a mismatched common name or are self signed.
 	
 		.EXAMPLE
 			PS C:\> Get-WebsiteCertificate "https://www.gmail.com" -UseSystemProxy -UseDefaultCredentials -TrustAllCertificates -OutputFile C:\gmail.cer
 		
 		.INPUTS
 			Does not accept pipeline input.
 	
 		.OUTPUTS
 			System.Security.Cryptography.X509Certificates.X509Certificate, System.IO.FileInfo
 	#>
 }
 Get-WebsiteCertificate "https://$vCACAPFQDN" -TrustAllCertificates -outputfile "C:\Temp\AP.cer"
`

