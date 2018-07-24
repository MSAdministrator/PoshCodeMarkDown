---
Author: vadims podans
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1793
Published Date: 2012-04-19t10
Archived Date: 2012-06-18t13
---

# create-certificate - 

## Description

creates self-signed signing certificate and installs it to certificate store.

## Comments



## Usage



## TODO



## function

`new-signingcert`

## Code

`#
 #
 #####################################################################
 #
 #
 #
 #####################################################################
 
 function New-SigningCert {
 <#
 .Synopsis
 	Creates self-signed signing certificate and install it to certificate store
 .Description
 	This function generates self-signed certificate with some pre-defined and
 	user-definable settings. User may elect to perform complete certificate
 	installation, by installing generated certificate to Trusted Root Certification
 	Authorities and Trusted Publishers containers in *current user* store.
 	
 .Parameter Subject
 	Specifies subject for certificate. This parameter must be entered in X500
 	Distinguished Name format. Default is: CN=PowerShell User, OU=Test Signing Cert.
 
 .Parameter KeyLength
 	Specifies private key length. Due of performance and security reasons, only
 	1024 and 2048 bit are supported. by default 1024 bit key length is used.
 
 .Parameter NotBefore
 	Sets the date in local time on which a certificate becomes valid. By default
 	current date and time is used.
 
 .Parameter NotAfter
 	Sets the date in local time after which a certificate is no longer valid. By
 	default certificate is valid for 365 days.
 
 .Parameter Force
 	If Force switch is asserted, script will prepare certificate for use by adding
 	it to Trusted Root Certification Authorities and Trusted Publishers containers
 	in current user certificate store. During certificate installation you will be
 	prompted to confirm if you want to add self-signed certificate to Trusted Root
 	Certification Authorities container.
 #>
 [CmdletBinding()]
 	param (
 		[string]$Subject = "CN=PowerShell User, OU=Test Signing Cert",
 		[int][ValidateSet("1024", "2048")]$KeyLength = 1024,
 		[datetime]$NotBefore = [DateTime]::Now,
 		[datetime]$NotAfter = $NotBefore.AddDays(365),
 		[switch]$Force
 	)
 	
 	$OS = (Get-WmiObject Win32_OperatingSystem).Version
 	if ($OS[0] -lt 6) {
 		Write-Warning "Windows XP, Windows Server 2003 and Windows Server 2003 R2 are not supported!"
 		return
 	}
 	
 	$SubjectDN = New-Object -ComObject X509Enrollment.CX500DistinguishedName
 	$SubjectDN.Encode($Subject, 0x0)
 	
 	$OID = New-Object -ComObject X509Enrollment.CObjectID
 	$OID.InitializeFromValue("1.3.6.1.5.5.7.3.3")
 	$OIDs = New-Object -ComObject X509Enrollment.CObjectIDs
 	$OIDs.Add($OID)
 	
 	$EKU = New-Object -ComObject X509Enrollment.CX509ExtensionEnhancedKeyUsage
 	$EKU.InitializeEncode($OIDs)
 	
 	$PrivateKey = New-Object -ComObject X509Enrollment.CX509PrivateKey
 	$PrivateKey.ProviderName = "Microsoft Base Cryptographic Provider v1.0"
 	$PrivateKey.KeySpec = 0x2
 	$PrivateKey.Length = $KeyLength
 	$PrivateKey.MachineContext = 0x0
 	$PrivateKey.Create()
 	
 	$Cert = New-Object -ComObject X509Enrollment.CX509CertificateRequestCertificate
 	$Cert.InitializeFromPrivateKey(0x1,$PrivateKey,"")
 	$Cert.Subject = $SubjectDN
 	$Cert.Issuer = $Cert.Subject
 	$Cert.NotBefore = $NotBefore
 	$Cert.NotAfter = $NotAfter
 	$Cert.X509Extensions.Add($EKU)
 	$Cert.Encode()
 	
 	
 	$Request = New-Object -ComObject X509Enrollment.CX509enrollment
 	$Request.InitializeFromRequest($Cert)
 	$endCert = $Request.CreateRequest(0x1)
 	$Request.InstallResponse(0x2,$endCert,0x1,"")
 	
 	if ($Force) {
 	 	[Byte[]]$bytes = [System.Convert]::FromBase64String($endCert)
 		foreach ($Container in "Root", "TrustedPublisher") {
 			$x509store = New-Object Security.Cryptography.X509Certificates.X509Store $Container, "CurrentUser"
 			$x509store.Open([Security.Cryptography.X509Certificates.OpenFlags]::ReadWrite)
 			$x509store.Add([Security.Cryptography.X509Certificates.X509Certificate2]$bytes)
 			$x509store.Close()
 		}
 	}
 }
`

