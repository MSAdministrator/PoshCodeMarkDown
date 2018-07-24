---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5547
Published Date: 2014-10-29t06
Archived Date: 2014-11-04t04
---

# ping-wsman - 

## Description

it’s like test-wsman but you can set the flags, so you can turn off cacheck and cncheck (and i do, by default)

## Comments



## Usage



## TODO



## module

`ping-wsman`

## Code

`#
 #
 Import-Module Microsoft.WSMan.Management
 Add-Type @"
   [System.Flags]
   public enum WSManSessionFlags {
     None = 0, UseUtf8 = 1, SkipCACheck = 8192, SkipCNCheck = 16384, UseSsl = 134217728,
     UseNoAuthentication = 32768, 
     Negotiate = 131072, Kerberos = 524288, ClientCertificate = 2097152, 
     Digest = 65536, Basic = 262144, CredSsp = 16777216, CredUserNamePassword = 4096, 
     
     NoEncryption = 1048576,  EnableSpnServerPort = 4194304, UseUtf16 = 8388608, 
     SkipRevocationCheck = 33554432, AllowNegotiateImplicitCredentials = 67108864
   }
 
   [System.Flags]
   public enum WSManSessionOptions {
     None = 0, UseUtf8 = 1, UseUtf16 = 8388608, 
     SkipCACheck = 8192, SkipCNCheck = 16384, UseSsl = 134217728,
     NoEncryption = 1048576,  EnableSpnServerPort = 4194304, 
     SkipRevocationCheck = 33554432, AllowNegotiateImplicitCredentials = 67108864    
   }
 
   [System.Flags]
   public enum WSManAuthentication {
     None = 32768, Basic = 266240, Negotiate = 131072, Kerberos = 524288, 
     ClientCertificate = 2097152, Digest = 69632, CredSsp = 16781312
   }
 "@
 
 function Ping-WsMan {
     #.Synopsis
     #.Description
     [CmdletBinding(DefaultParameterSetName="ComputerName")]
     param(
         [Parameter(ParameterSetName="ComputerName")]
         [Alias("CN")]
         $ComputerName = 'localhost',
 
         [Parameter(ParameterSetName="ComputerName")]
         [int]$Port = 5985,
 
         [Parameter(ParameterSetName="ComputerName")]
         [string]$ApplicationName,
         
         [Parameter(ParameterSetName="Url")]
         [string]$Uri = "${ComputerName}:${Port}" + $( if($ApplicationName){ "/$ApplicationName" } ),
 
         [Switch]$UseSsl,
 
         [WSManSessionOptions]$SessionOptions = ("SkipCACheck","SkipCNCheck"),
 
         [WSManAuthentication]$Authentication = "None",
 
         [System.Management.Automation.Credential()]
         [pscredential]$Credential
     )
 
     if($UseSsl) {
         if(!$Port){ $Port = 5986 }
         $SessionOptions += "UseSsl"
         $Uri = "${ComputerName}:${Port}" + $( if($ApplicationName){ "/$ApplicationName" } )
     }
 
     [WSManSessionFlags]$SessionFlags = $SessionOptions -bor $Authentication
     $wsman = New-Object Microsoft.WSMan.Management.WSManClass
     
     if($Credential) {
         if($Authentication -eq "None") {
             $SessionFlags -= "UseNoAuthentication"
             $SessionFlags += "CredUserNamePassword"
         }
         $options = [System.__ComObject].InvokeMember('CreateConnectionOptions', 'InvokeMethod', $null, $wsman, $null)
         $options.UserName = $Credential.UserName
         $options.Password = $Credential.GetNetworkCredential().Password
         $Parameters = $Uri, $SessionFlags, $options
     } else {
         $Parameters = $Uri, $SessionFlags
     }
 
     Write-Verbose $SessionFlags
 
     $session = [System.__ComObject].InvokeMember('CreateSession', 'InvokeMethod', $null, $wsman, $Parameters)
      
     $([Xml]$session.Identify(0)).DocumentElement
 }
`

