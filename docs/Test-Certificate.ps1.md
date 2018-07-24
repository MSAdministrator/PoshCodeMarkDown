---
Author: vpodans
Publisher: 
Copyright: 
Email: 
Version: 0.9
Encoding: ascii
License: cc0
PoshCode ID: 1634
Published Date: 
Archived Date: 2010-10-15t20
---

# test-certificate - 

## Description

tests specified certificate for certificate chain and revocation

## Comments



## Usage



## TODO



## function

`test-certificate`

## Code

`#
 #
 #####################################################################
 #
 #
 #####################################################################
 
 function Test-Certificate {
 <#
 .Synopsis
 	Tests specified certificate for certificate chain and revocation
 .Description
 	Tests specified certificate for certificate chain and revocation status for each certificate in chain
 	exluding Root certificates
 .Parameter Certificate
 	Specifies the certificate to test certificate chain. This parameter may accept
 	X509Certificate, X509Certificate2 objects or physical file path. this paramter accept
 	pipeline input
 .Parameter Password
 	Specifies PFX file password. Password must be passed as SecureString.
 .Parameter CRLMode
 	Sets revocation check mode. May contain on of the following values:
 	
 	Online - perform revocation check downloading CRL from CDP extension ignoring cached CRLs. Default value
 	Offline - perform revocation check using cached CRLs if they are already downloaded
 	NoCheck - specified certificate will not checked for revocation status (not recommended)
 .Parameter CRLFlag
 	Sets revocation flags for chain elements. May contain one of the following values:
 	
 	ExcludeRoot - perform revocation check for each certificate in chain exluding root. Default value
 	EntireChain - perform revocation check for each certificate in chain including root. (not recommended)
 	EndCertificateOnly - perform revocation check for specified certificate only.
 .Parameter VerificationFlags
 	Sets verification checks that will bypassed performed during certificate chaining engine
 	check. You may specify one of the following values:
 	
 	NoFlag - No flags pertaining to verification are included (default).
 	IgnoreNotTimeValid - Ignore certificates in the chain that are not valid either because they have expired or they
 		are not yet	in effect when determining certificate validity.
 	IgnoreCtlNotTimeValid - Ignore that the certificate trust list (CTL) is not valid, for reasons such as the CTL
 		has expired, when determining certificate verification.
 	IgnoreNotTimeNested - Ignore that the CA (certificate authority) certificate and the issued certificate have
 		validity periods that are not nested when verifying the certificate. For example, the CA cert can be valid
 		from January 1 to December 1 and the issued certificate from January 2 to December 2, which would mean the
 		validity periods are not nested.
 	IgnoreInvalidBasicConstraints - Ignore that the basic constraints are not valid when determining certificate
 		verification.
 	AllowUnknownCertificateAuthority - Ignore that the chain cannot be verified due to an unknown certificate
 		authority (CA).
 	IgnoreWrongUsage - Ignore that the certificate was not issued for the current use when determining
 		certificate verification.
 	IgnoreInvalidName - Ignore that the certificate has an invalid name when determining certificate verification.
 	IgnoreInvalidPolicy - Ignore that the certificate has invalid policy when determining certificate verification.
 	IgnoreEndRevocationUnknown - Ignore that the end certificate (the user certificate) revocation is unknown when
 		determining	certificate verification.
 	IgnoreCtlSignerRevocationUnknown - Ignore that the certificate trust list (CTL) signer revocation is unknown
 		when determining certificate verification.
 	IgnoreCertificateAuthorityRevocationUnknown - Ignore that the certificate authority revocation is unknown 
 		when determining certificate verification.
 	IgnoreRootRevocationUnknown - Ignore that the root revocation is unknown when determining certificate verification.
 	AllFlags - All flags pertaining to verification are included.	
 .Example
 	Get-ChilItem cert:\CurrentUser\My | Test-Certificate -CRLMode "NoCheck"
 	
 	Will check certificate chain for each certificate in current user Personal container.
 	Certificates will not checked for revocation status.
 .Example
 	Test-Certificate C:\Certs\certificate.cer -CRLFlag "EndCertificateOnly"
 	
 	Will check certificate chain for certificate that is located in C:\Certs and named
 	as Certificate.cer and revocation checking will be performed for specified certificate only
 .Outputs
 	This script return general info about certificate chain status
 #>
 [CmdletBinding()]
 	param(
 		[Parameter(Mandatory = $true, ValueFromPipeline = $true, Position = 0)]
 		$Certificate,
 		[System.Security.SecureString]$Password,
 		[System.Security.Cryptography.X509Certificates.X509RevocationMode]$CRLMode = "Online",
 		[System.Security.Cryptography.X509Certificates.X509RevocationFlag]$CRLFlag = "ExcludeRoot",
 		[System.Security.Cryptography.X509Certificates.X509VerificationFlags]$VerificationFlags = "NoFlag"
 	)
 	
 	begin {
 		$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
 		$chain = New-Object System.Security.Cryptography.X509Certificates.X509Chain
 		$chain.ChainPolicy.RevocationFlag = $CRLFlag
 		$chain.ChainPolicy.RevocationMode = $CRLMode
 		$chain.ChainPolicy.VerificationFlags = $VerificationFlags
 		function _getstatus_ ($status, $chain, $cert) {
 			if ($status) {
 				Write-Host Current certificate $cert.SerialNumber chain and revocation status is valid -ForegroundColor Green
 			} else {
 				Write-Warning "Current certificate $($cert.SerialNumber) chain is invalid due of the following errors:"
 				$chain.ChainStatus | %{Write-Host $_.StatusInformation.trim() -ForegroundColor Red}
 			}
 		}
 	}
 	
 	process {
 		if ($_ -is [System.Security.Cryptography.X509Certificates.X509Certificate2]) {
 			$status = $chain.Build($_)
 			_getstatus_ $status $chain $_
 		} else {
 			if (!(Test-Path $Certificate)) {Write-Warning "Specified path is invalid"; return}
 			else {
 				if ((Resolve-Path $Certificate).Provider.Name -ne "FileSystem") {
 					Write-Warning "Spicifed path is not recognized as filesystem path. Try again"; return
 				} else {
 					$Certificate = gi $(Resolve-Path $Certificate)
 					switch -regex ($Certificate.Extension) {
 					"\.CER|\.DER|\.CRT" {$cert.Import($Certificate.FullName)}
 					"\.PFX" {
 						if (!$Password) {$Password = Read-Host "Enter password for PFX file $certificate" -AsSecureString}
 							$cert.Import($Certificate.FullName, $password, "UserKeySet")
 						}
 					"\.P7B|\.SST" {
 						$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2Collection
 						$cert.Import([System.IO.File]::ReadAllBytes($Certificate.FullName))
 						}
 					default {Write-Warning "Looks like your specified file is not a certificate file"; return}
 					}
 					$cert | %{
 						$status = $chain.Build($_)
 						_getstatus_ $status $chain $_
 					}
 					$cert.Reset()
 					$chain.Reset()
 				}
 			}
 		}
 	}
 }
`

