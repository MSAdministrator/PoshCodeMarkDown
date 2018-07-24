---
Author: pavel
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5586
Published Date: 2016-11-13t17
Archived Date: 2016-03-14t10
---

# minor updated version - 

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
 [CmdletBinding(PositionalBinding=$false)] 
     param (
 
 	    [Parameter(
 		    HelpMessage = "Enter the common name (or subject) of the Certificate",
 		    Mandatory = $true,
 		    ValueFromPipelineByPropertyName = $true
 	    )]
 	    [ValidateNotNullOrEmpty()]
 	    [String]$CN,
 
         [Parameter(Mandatory=$False)] [string] $OU = "My OU",
 	    [Parameter(Mandatory=$False)] [string] $O = "My Organisation",
 	    [Parameter(Mandatory=$False)] [string] $L = "My locality",
 	    [Parameter(Mandatory=$False)] [string] $S = "My state" ,
 	    [Parameter(Mandatory=$False)] [string] $C = "My country",
 	    [Parameter(Mandatory=$False)] [string] $E = "My email",
 	    [Parameter(
 		    HelpMessage = "Enter DNS Subject Alternative Names", 
 		    Mandatory = $false,
 		    ValueFromPipelineByPropertyName = $true 
 	    )]
 	    [ValidateNotNullOrEmpty()]	    
         [string[]] $DNSNames,
         [Parameter(
 		    HelpMessage = "Enter RFC822 Subject Alternative Names", 
 		    Mandatory = $false,
 		    ValueFromPipelineByPropertyName = $true 
 	    )]
 	    [ValidateNotNullOrEmpty()]
         [string[]] $RFC822Names,
         [Parameter(Mandatory=$False)] [string] $FriendlyName,
         [Parameter(Mandatory=$False)] [string] $Description,
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
 	    [String[]] $ExtendedKeyUsage = ("Server Authentication", "Client Authentication"),
 	    [Parameter( 
 		    Mandatory = $false 
 	    )]
 	    [ValidateNotNullOrEmpty()]
 	    [string] $ProviderName = ("Microsoft RSA SChannel Cryptographic Provider"),
 
 	    [Parameter( 
 		    Mandatory = $false 
 	    )]
 	    [ValidateSet(
 		    "sha256", "sha384", "sha512", "sha1", "md5", "md4", "md2"
 	    )]
 	    [string] $HashAlgorithm = "sha256",
         [Parameter(Mandatory=$False)] [int] [ValidateSet(1024,2048,4096)] $KeySize = 2048,
 	    [Parameter(Mandatory=$false)] [string] $SaveDirectoryPath = $(Get-Location),
         [switch] $SaveOutput
     )
 begin
 {
     $ErrorActionPreference = "Stop"
 
     $AlternativeNameType = @{
         XCN_CERT_ALT_NAME_UNKNOWN = 0
         XCN_CERT_ALT_NAME_OTHER_NAME = 1
         XCN_CERT_ALT_NAME_RFC822_NAME = 2
         XCN_CERT_ALT_NAME_DNS_NAME = 3
         XCN_CERT_ALT_NAME_DIRECTORY_NAME = 5
         XCN_CERT_ALT_NAME_URL = 7
         XCN_CERT_ALT_NAME_IP_ADDRESS = 8
         XCN_CERT_ALT_NAME_REGISTERED_ID = 9
         XCN_CERT_ALT_NAME_GUID = 10
         XCN_CERT_ALT_NAME_USER_PRINCIPLE_NAME = 11
     }
 
     $ObjectIdGroupId = @{
         XCN_CRYPT_ANY_GROUP_ID = 0
         XCN_CRYPT_HASH_ALG_OID_GROUP_ID = 1
         XCN_CRYPT_ENCRYPT_ALG_OID_GROUP_ID = 2
         XCN_CRYPT_PUBKEY_ALG_OID_GROUP_ID = 3
         XCN_CRYPT_SIGN_ALG_OID_GROUP_ID = 4
         XCN_CRYPT_RDN_ATTR_OID_GROUP_ID = 5
         XCN_CRYPT_EXT_OR_ATTR_OID_GROUP_ID = 6
         XCN_CRYPT_ENHKEY_USAGE_OID_GROUP_ID = 7
         XCN_CRYPT_POLICY_OID_GROUP_ID = 8
         XCN_CRYPT_TEMPLATE_OID_GROUP_ID = 9
         XCN_CRYPT_LAST_OID_GROUP_ID = 9
         XCN_CRYPT_FIRST_ALG_OID_GROUP_ID = 1
         XCN_CRYPT_LAST_ALG_OID_GROUP_ID = 4
         XCN_CRYPT_OID_DISABLE_SEARCH_DS_FLAG = 0x80000000
         XCN_CRYPT_KEY_LENGTH_MASK = 0xffff0000
     }
 
     $X509KeySpec = @{
     }
 
     $X509PrivateKeyExportFlags = @{
         XCN_NCRYPT_ALLOW_EXPORT_NONE = 0
         XCN_NCRYPT_ALLOW_EXPORT_FLAG = 0x1
         XCN_NCRYPT_ALLOW_PLAINTEXT_EXPORT_FLAG = 0x2
         XCN_NCRYPT_ALLOW_ARCHIVING_FLAG = 0x4
         XCN_NCRYPT_ALLOW_PLAINTEXT_ARCHIVING_FLAG = 0x8
     }
 
     $X509PrivateKeyUsageFlags = @{
         XCN_NCRYPT_ALLOW_USAGES_NONE = 0
         XCN_NCRYPT_ALLOW_DECRYPT_FLAG = 0x1
         XCN_NCRYPT_ALLOW_SIGNING_FLAG = 0x2
         XCN_NCRYPT_ALLOW_KEY_AGREEMENT_FLAG = 0x4
         XCN_NCRYPT_ALLOW_ALL_USAGES = 0xffffff
     }
 
     $X509CertificateEnrollmentContext = @{
         ContextUser = 0x1
         ContextMachine = 0x2
         ContextAdministratorForceMachine = 0x3
     }
 
     $X509KeyUsageFlags = @{
     }
 
     $EncodingType = @{
         XCN_CRYPT_STRING_BASE64HEADER = 0
         XCN_CRYPT_STRING_BASE64 = 0x1
         XCN_CRYPT_STRING_BINARY = 0x2
         XCN_CRYPT_STRING_BASE64REQUESTHEADER = 0x3
         XCN_CRYPT_STRING_HEX = 0x4
         XCN_CRYPT_STRING_HEXASCII = 0x5
         XCN_CRYPT_STRING_BASE64_ANY = 0x6
         XCN_CRYPT_STRING_ANY = 0x7
         XCN_CRYPT_STRING_HEX_ANY = 0x8
         XCN_CRYPT_STRING_BASE64X509CRLHEADER = 0x9
         XCN_CRYPT_STRING_HEXADDR = 0xa
         XCN_CRYPT_STRING_HEXASCIIADDR = 0xb
         XCN_CRYPT_STRING_HEXRAW = 0xc
         XCN_CRYPT_STRING_NOCRLF = 0x40000000
         XCN_CRYPT_STRING_NOCR = 0x80000000
     }
 
     $InstallResponseRestrictionFlags = @{
         AllowNone = 0x00000000
         AllowNoOutstandingRequest = 0x00000001
         AllowUntrustedCertificate = 0x00000002
         AllowUntrustedRoot = 0x00000004
     }
 
     $X500NameFlags = @{
         XCN_CERT_NAME_STR_NONE = 0
         XCN_CERT_SIMPLE_NAME_STR = 1
         XCN_CERT_OID_NAME_STR = 2
         XCN_CERT_X500_NAME_STR = 3
         XCN_CERT_XML_NAME_STR = 4
         XCN_CERT_NAME_STR_SEMICOLON_FLAG = 0x40000000
         XCN_CERT_NAME_STR_NO_PLUS_FLAG = 0x20000000
         XCN_CERT_NAME_STR_NO_QUOTING_FLAG = 0x10000000
         XCN_CERT_NAME_STR_CRLF_FLAG = 0x8000000
         XCN_CERT_NAME_STR_COMMA_FLAG = 0x4000000
         XCN_CERT_NAME_STR_REVERSE_FLAG = 0x2000000
         XCN_CERT_NAME_STR_FORWARD_FLAG = 0x1000000
         XCN_CERT_NAME_STR_DISABLE_IE4_UTF8_FLAG = 0x10000
         XCN_CERT_NAME_STR_ENABLE_T61_UNICODE_FLAG = 0x20000
         XCN_CERT_NAME_STR_ENABLE_UTF8_UNICODE_FLAG = 0x40000
         XCN_CERT_NAME_STR_FORCE_UTF8_DIR_STR_FLAG = 0x80000
         XCN_CERT_NAME_STR_DISABLE_UTF8_DIR_STR_FLAG = 0x100000
     }
 
     $ObjectIdPublicKeyFlags = @{
         XCN_CRYPT_OID_INFO_PUBKEY_ANY = 0
         XCN_CRYPT_OID_INFO_PUBKEY_SIGN_KEY_FLAG = 0x80000000
         XCN_CRYPT_OID_INFO_PUBKEY_ENCRYPT_KEY_FLAG = 0x40000000
     }
 
     $AlgorithmFlags = @{
         AlgorithmFlagsNone = 0
         AlgorithmFlagsWrap = 0x1
     }
 }
     process
     {
         $dn = "CN={0}, OU={1}, O={2}, L={3}, S={4}, C={5}, E={6}" -f $CN, $OU, $O, $L, $S, $C, $E
 
         Write-Verbose "Creating CSR for certificate with DN=$dn"
 
         If (-not ($SaveOutput))
         {
             Write-Verbose """SaveOuput"" parameter has not been specified therefore the output will not be saved"
         }
 
         if ($DNSNames)
 	    {
 		    [string[]]$DNSNames += $CN
 	    }
 
 	    [String]$KeyUsage = $KeyUsage -join ","
 
 	   
         $objDN = New-Object -ComObject X509Enrollment.CX500DistinguishedName
         $objDN.Encode($dn, $X500NameFlags.XCN_CERT_NAME_STR_NONE)
 
 
 	    $PrivateKey = New-Object -ComObject X509Enrollment.CX509PrivateKey -Property @{
 		    ProviderName = $ProviderName
 		    MachineContext = $true
 		    Length = $KeySize
 		    KeySpec =  $X509KeySpec.XCN_AT_KEYEXCHANGE
 		    KeyUsage = ([int][Security.Cryptography.X509Certificates.X509KeyUsageFlags]($KeyUsage))
 		    ExportPolicy = $X509PrivateKeyExportFlags.XCN_NCRYPT_ALLOW_EXPORT_FLAG
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
 
 	    $PKCS10.InitializeFromPrivateKey($X509CertificateEnrollmentContext.ContextMachine, $PrivateKey, "")
 	    $PKCS10.Subject = $objDN
 
 	    
 	    if (($DNSNames) -or ($RFC822Names))
 	    {
 		    $IAlternativeNames = New-Object -ComObject X509Enrollment.CAlternativeNames
 
 		    foreach ($DNSName in $DNSNames) 
 		    {
 			    $DNSAltName = New-Object -ComObject X509Enrollment.CAlternativeName
 
 			    $DNSAltName.InitializeFromString($AlternativeNameType.XCN_CERT_ALT_NAME_DNS_NAME,$DNSName)
 
 			    $IAlternativeNames.Add($DNSAltName)
 		    }
 
             foreach ($RFC822Name in $RFC822Names) 
 		    {
                
 			    $RFC822AltName = New-Object -ComObject X509Enrollment.CAlternativeName
 
 			    $RFC822AltName.InitializeFromString($AlternativeNameType.XCN_CERT_ALT_NAME_RFC822_NAME,$RFC822Name)
 
 			    $IAlternativeNames.Add($RFC822AltName)
             }
             
 		    $SubjectAlternativeName = New-Object -ComObject X509Enrollment.CX509ExtensionAlternativeNames
 		    $SubjectAlternativeName.InitializeEncode($IAlternativeNames)
             $PKCS10.X509Extensions.Add($SubjectAlternativeName)
 	    }
 
 
 	    $PKCS10.X509Extensions.Add($ExtendedKeyUsageObj)
 	    $PKCS10.X509Extensions.Add($KeyUsageObj)
 
 		$HashObjectId = New-Object -ComObject X509Enrollment.CObjectID
 		$HashObjectId.InitializeFromValue(([Security.Cryptography.Oid]($HashAlgorithm)).Value)
 		$PKCS10.HashAlgorithm = $HashObjectId
 
 
 	    $Request = New-Object -ComObject X509Enrollment.CX509Enrollment
 
 	    $Request.CertificateFriendlyName = $FriendlyName
 	    $Request.CertificateDescription = $Description
 
 	    $Request.InitializeFromRequest($PKCS10)
 
         $CSR = $Request.CreateRequest($EncodingType.XCN_CRYPT_STRING_BASE64HEADER)
         
         Write-Output -InputObject $CSR
         
         If ($SaveOutput)
         {
             If (-not(Test-Path -Path $SaveDirectoryPath -PathType Container))
             {
                 $null = New-Item -Path $SaveDirectoryPath -ItemType directory    
             }
             
             $SaveFilePath = Join-Path -Path $SaveDirectoryPath -ChildPath $("{0}-{1}.req" -f $($CN.Replace("*","_")), (Get-Date -Format "yyyy-MMM-dd_HH-mm-ss"))
             $null = New-Item -Path $SaveFilePath -ItemType file
             Set-Content -Path $SaveFilePath -Value ($CSR) -Encoding ASCII
 
             Write-Verbose "CSR saved to file path ""$SaveFilePath"""
         }
     }
`

