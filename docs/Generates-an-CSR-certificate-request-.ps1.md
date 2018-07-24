---
Author: pavel_kandratsyeu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5400
Published Date: 
Archived Date: 2016-02-14t00
---

#  - 

## Description

generates an csr certificate request powershell

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 
 .DESCRIPTION
 Offline request is the request that is saved to a file and is submitted to CA server by using out of band means. 
 Once the certificate is issued, it is installed on the original computer.
 
 .LINK
 http://en-us.sysadmins.lv/Lists/Posts/Post.aspx?List=332991f0-bfed-4143-9eea-f521167d287c&ID=74
 http://msdn.microsoft.com/en-us/library/aa374863(v=vs.85).aspx
 http://www.css-security.com/blog/creating-a-self-signed-ssl-certificate-using-powershell/
 
 .NOTES 
 Created: 02-Sept-2014
 Updated: 03-Sept-2014
 Author: Pavel_Kandratsyeu@epam.com
 #>
 
 [CmdletBinding()] 
 param (
 
 	[Parameter(
 		HelpMessage = "Enter Subject of Certificate e.g. *.qa.infogen.cc",
 		Mandatory = $true,
 		Position = 0, 
 		ValueFromPipelineByPropertyName = $true
 	)]
 	[ValidateNotNullOrEmpty()]
 	[String]$SubjectName,
 
 	[Parameter(
 		HelpMessage = "Enter Subject Alternative Name e.g. *.owa.epam.com", 
 		Mandatory = $false,
 		ValueFromPipelineByPropertyName = $true 
 	)]
 	[ValidateNotNullOrEmpty()]
 	[string[]]$AlternativeName,
 
 	[Parameter(
 		HelpMessage = "Enter Key Usage Extension e.g. 'DigitalSignature', 'KeyEncipherment'",
 		Mandatory = $false, 
 		ValueFromPipelineByPropertyName = $true
 	)]
 	[ValidateSet(
 		"None", "KeyEncipherment", "EncipherOnly", "CrlSign", "KeyCertSign", "KeyAgreement", 
 		"DataEncipherment", "KeyEncipherment", "NonRepudiation", "DigitalSignature", "DecipherOnly"
 	)]
 	[String[]]$KeyUsage = ("DigitalSignature", "KeyEncipherment"),
 
 	[Parameter(
 		HelpMessage = "Enter Extended Key Usage (application policies) e.g. 'Server Authentication', 'Client Authentication'",
 		Mandatory = $false, 
 		ValueFromPipelineByPropertyName = $true
 	)]
 	[ValidateSet(
 		"Time Stamping", "Microsoft Trust List Signing", "Microsoft Time Stamping", "IP security end system", "IP security tunnel termination", "IP security user",
 		"Encrypting File System", "Windows Hardware Driver Verification", "Windows System Component Verification", "OEM Windows System Component Verification",
 		"Embedded Windows System Component Verification", "Key Pack Licenses", "License Server Verification", "Smart Card Logon", "Digital Rights", "Qualified Subordination",
 		"Key Recovery", "Document Signing", "IP security IKE intermediate", "File Recovery", "Root List Signer", "All application policies",
 		"Directory Service Email Replication", "Certificate Request Agent", "Key Recovery Agent", "Private Key Archival", "Lifetime Signing", "OCSP Signing", "Any Purpose",
 		"KDC Authentication", "Kernel Mode Code Signing", "CTL Usage", "Revoked List Signer", "Early Launch Antimalware Driver", "Disallowed List", "HAL Extension",
 		"Endorsement Key Certificate", "Platform Certificate", "Attestation Identity Key Certificate", "Windows Kits Component", "Windows RT Verification",
 		"Protected Process Light Verification", "Windows TCB Component", "Protected Process Verification", "Windows Store", "Dynamic Code Generator", "Microsoft Publisher",
 		"Windows Third Party Application Component", "Windows Software Extension Verification", "System Health Authentication", "Domain Name System (DNS) Server Trust", 
 		"BitLocker Drive Encryption", "BitLocker Data Recovery Agent", "Windows Update"
 	)]
 	[String[]]$ExtendedKeyUsage = ("Server Authentication", "Client Authentication"),
 
 	[Parameter( 
 		Mandatory = $false 
 	)]
 	[ValidateNotNullOrEmpty()]
 	[string]$ProviderName = ("Microsoft RSA SChannel Cryptographic Provider"),
 
 	[Parameter( 
 		Mandatory = $false 
 	)]
 	[ValidateSet(
 		"sha256", "sha384", "sha512", "sha1", "md5", "md4", "md2"
 	)]
 	[string]$HashAlgorithm,
 
 
 	[Parameter( 
 		Mandatory = $false 
 	)]
 	[ValidateNotNullOrEmpty()]
 	[string]$OutputPath = ([IO.Path]::ChangeExtension($MyInvocation.MyCommand.Path, ".req"))
 )
 
 process
 {
 	if ($AlternativeName)
 	{
 		[string[]]$AlternativeName += $SubjectName
 	}
 
 	[String]$KeyUsage = $KeyUsage -join ","
 
 	[string]$Subject = "CN=$SubjectName,C=US,L=Newtonw,O=EPAM Systems,S=Pennsylvania"
 
 
 	$SubjectObj = New-Object -ComObject X509Enrollment.CX500DistinguishedName
 	$SubjectObj.Encode($Subject, 0x0)
 
 	if ($AlternativeName)
 	{
 		$IAlternativeNames = New-Object -ComObject X509Enrollment.CAlternativeNames
 
 		foreach ($AN in $AlternativeName) 
 		{
 			$AltName = New-Object -ComObject X509Enrollment.CAlternativeName
 
 			$AltName.InitializeFromString(0x3,$AN)
 
 			$IAlternativeNames.Add($AltName)
 		}
 
 		$SubjectAlternativeName = New-Object -ComObject X509Enrollment.CX509ExtensionAlternativeNames
 		$SubjectAlternativeName.InitializeEncode($IAlternativeNames)
 	}
 
 
 
 	$PrivateKey = New-Object -ComObject X509Enrollment.CX509PrivateKey -Property @{
 		ProviderName = $ProviderName
 		MachineContext = $true
 		Length = 2048
 		KeySpec = 1
 		KeyUsage = ([int][Security.Cryptography.X509Certificates.X509KeyUsageFlags]($KeyUsage))
 		ExportPolicy = 0x00000001
 	}
 
 	$PrivateKey.Create()
 
 	$KeyUsageObj = New-Object -ComObject X509Enrollment.CX509ExtensionKeyUsage
 	$KeyUsageObj.InitializeEncode([int][Security.Cryptography.X509Certificates.X509KeyUsageFlags]($KeyUsage))
 	$KeyUsageObj.Critical = $true
 
 
 
 	$ExtendedKeyUsageObj = New-Object -ComObject X509Enrollment.CX509ExtensionEnhancedKeyUsage
 	$ExtendedKeyUsageObj.Critical = $false
 
 	$ObjectIDs = New-Object -ComObject X509Enrollment.CObjectIDs
 
 	foreach ($EKU in $ExtendedKeyUsage)
 	{
 		$CryptoObjectId = [Security.Cryptography.Oid]($EKU)
 		$ObjectId = New-Object -ComObject X509Enrollment.CObjectID
 
 		$ObjectId.InitializeFromValue($CryptoObjectId.Value)
 
 		$ObjectIDs.Add($ObjectId)
 	}
 
 	$ExtendedKeyUsageObj.InitializeEncode($ObjectIDs)
 
 
 
 	$PKCS10 = New-Object -ComObject X509Enrollment.CX509CertificateRequestPkcs10
 
 	$PKCS10.InitializeFromPrivateKey(0x2, $PrivateKey, "")
 	$PKCS10.Subject = $SubjectObj
 
 	if ($AlternativeName)
 	{
 		$PKCS10.X509Extensions.Add($SubjectAlternativeName)
 	}
 
 	$PKCS10.X509Extensions.Add($ExtendedKeyUsageObj)
 	$PKCS10.X509Extensions.Add($KeyUsageObj)
 
 	if ($HashAlgorithm)
 	{
 		$HashObjectId = New-Object -ComObject X509Enrollment.CObjectID
 		$HashObjectId.InitializeFromValue(([Security.Cryptography.Oid]($HashAlgorithm)).Value)
 		$PKCS10.HashAlgorithm = $HashObjectId
 	}
 
 
 	$Request = New-Object -ComObject X509Enrollment.CX509Enrollment
 
 	$Request.CertificateFriendlyName = $SubjectName
 	$Request.CertificateDescription = $SubjectName
 
 	$Request.InitializeFromRequest($PKCS10)
 
 	Set-Content -Path $OutputPath -Value ($Request.CreateRequest(0x3)) -Encoding ASCII
 }
`

