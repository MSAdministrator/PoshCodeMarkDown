---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6426
Published Date: 2016-06-28t17
Archived Date: 2016-07-01t08
---

# dynamic parameters huh? - 

## Description

i can’t get dynamic parameters working when there are multiple parameter sets on the other parameters. as soon as i limit the param sets down to 1, it works. if there are 2, it doesn’t.

## Comments



## Usage



## TODO



## function

`dynamic-paramtest`

## Code

`#
 #
 function Dynamic-ParamTest {
     param(
         [parameter(Mandatory=$true)]
         [string]$Hostname,
         
         [parameter()]
         [string]$Protocol='Sftp',
 
         [parameter()]
 
         [parameter(Mandatory=$true)]
         [string]$Username,
     
         [parameter(Mandatory=$true,ParameterSetName="SecurePass")]
         [string]$PasswordFile,
 
         [string]$SshHostKeyFingerprint,
 
         [parameter()]
         [string]$TlsHostCertificateFingerprint,
 
         [parameter(ParameterSetName="SecurePass")]
         [switch]$SetSecurePassword,
 
         [parameter()]
         [string]$SshPrivateKeyPath,
 
         [parameter()]
         [string]$SshPrivateKeyPassphrase,
 
         [parameter()]
         [ValidateSet('None','Implicit','ExplicitSsl','Explicit','ExplicitTls')]
         [string]$FtpSecureMode = $null,
 
         [ValidateNotNullOrEmpty()]
         [string]$PlainTextPassword,
         
         [parameter()]
         [bool]$WebdavSecure,
 
         [parameter()]
         [string]$WebdavRoot,
 
         [parameter()]
         [string]$DLLFolder=$Script:WinSCPDLLFolder,
 
         [parameter()]
         [switch]$GiveUpSecurityAndAcceptAnySshHostKey,
 
         [parameter()]
         [switch]$GiveUpSecurityAndAcceptAnyTlsHostCertificate
     )
     
     DynamicParam{
         if ($Protocol -match "^Sftp$")
         {
             $attributes = new-object System.Management.Automation.ParameterAttribute
             $attributes.ParameterSetName = "__AllParameterSets"
             $attributes.Mandatory = $false
             $attributeCollection = new-object -Type System.Collections.ObjectModel.Collection[System.Attribute]
             $attributeCollection.Add($attributes)
 
             $dynParam1 = new-object -Type System.Management.Automation.RuntimeDefinedParameter("IAMDYNAMIC", [Int32], $attributeCollection)
             
             $paramDictionary = new-object -Type System.Management.Automation.RuntimeDefinedParameterDictionary
             $paramDictionary.Add("IAMDYNAMIC", $dynParam1)
             return $paramDictionary
         }
     }
 
     process{
         write-host "I Ran"
     }
 
 
 
 }
`

