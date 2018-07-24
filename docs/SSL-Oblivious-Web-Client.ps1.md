---
Author: stephen campbell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 634
Published Date: 2009-10-09t21
Archived Date: 2014-04-07t16
---

# ssl oblivious web client - 

## Description

this function creates a web client that will ignore all ssl certificate errors. useful for uploading (http put, maybe post as well) to an https web server using a self-signed cert.

## Comments



## Usage



## TODO



## function

`new-trustallwebclient`

## Code

`#
 #
 function New-TrustAllWebClient {
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
 
 	$WCSource=@'
 	  namespace Local.ToolkitExtensions.Net {
 	    class WebClient : System.Net.WebClient {
 	      protected override System.Net.WebRequest GetWebRequest(System.Uri uri) {
 	        System.Net.WebRequest webRequest = base.GetWebRequest(uri);
 	        webRequest.PreAuthenticate = true;
 	        webRequest.Timeout = System.Threading.Timeout.Infinite;
 	        return webRequest;
 	      }
 	    }
 	  }
 '@
 	$WCResults=$Provider.CompileAssemblyFromSource($Params,$WCSource)
 	$WCAssembly=$WCResults.CompiledAssembly
 
 	$WebClient=$WCAssembly.CreateInstance("Local.ToolkitExtensions.Net.WebClient")
 	return $WebClient
 }
 
`

