---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 2102
Published Date: 2013-08-25t07
Archived Date: 2013-03-05t02
---

# centurionportal - 

## Description

generates a self-signed certificate authority and a code-signing certificate using openssl

## Comments



## Usage



## TODO



## script

`get-useremail`

## Code

`#
 #
 ########################################################################################################################
 ########################################################################################################################
 ########################################################################################################################
 ##
 ##
 ##
 ##
 ##
 ########################################################################################################################
 ##
 Param(
 $CertStorageLocation = (join-path (split-path $Profile) "Certs"),
 $UserName       = (Read-Host "User name")
 
 , $email        
 , $company      
 , $department   
 , $city         
 , $state        
 , $country      
 
 , $RootCAName   = "Self-Signed-Root-CA"
 , $CodeSignName = "$UserName Code-Signing"
 , $alias        = "PoshCert",
 
 
 [string]$keyBits = 4096,
 [string]$days = 365,
 [string]$daysCA = (365 * 5),
 
 [switch]$forceNew = $false,
 [switch]$importall = $false,
 [switch]$import = ($false -or $importall),
 
 $CAPassword = $null,
 $CodeSignPassword = $null,
 
 $OpenSSLLocation = $null,
 $RootCAPassword = $Null, 
 $CodeSignCertPassword = $null
 )
 
 
 function Get-UserEmail {
    if(!$script:email) {
       $script:email = (Read-Host "Email address")
    }
    return $script:email
 }
 
 function Get-RootCAPassword {
    if(!$script:RootCAPassword) { 
       if(!$script:CAPassword) {
          $script:CAPassword = ((new-object System.Management.Automation.PSCredential "hi",(Read-Host -AsSecureString "Root CA Password")).GetNetworkCredential().Password)
       }
 
       $script:RootCAPassword = [Convert]::ToBase64String( (new-Object Security.Cryptography.PasswordDeriveBytes ([Text.Encoding]::UTF8.GetBytes($CaPassword)), ([Text.Encoding]::UTF8.GetBytes("$company$RootCAName")), "SHA1", 5).GetBytes(64) )
    }
    return $script:RootCAPassword
 }
 
 function Get-CodeSignPassword {
    if(!$script:CodeSignCertPassword) { 
     
       if(!$script:CodeSignPassword) {
          $script:CodeSignPassword = ((new-object System.Management.Automation.PSCredential "hi",(Read-Host -AsSecureString "Code Signing Password")).GetNetworkCredential().Password)
       }
       $script:CodeSignCertPassword = ([Convert]::ToBase64String( (new-Object Security.Cryptography.PasswordDeriveBytes ([Text.Encoding]::UTF8.GetBytes($CodeSignPassword)), ([Text.Encoding]::UTF8.GetBytes((Get-UserEmail))), "SHA1", 5).GetBytes(64) ))
    }
    return $script:CodeSignCertPassword
 }
 
 function Get-SslConfig {
 Param ( 
    $keyBits, 
    $Country    = (Read-Host "Country (2-Letter code)"), 
    $State      = (Read-Host "State (Full Name, no intials)"), 
    $city       = (Read-Host "City"), 
    $company    = (Read-Host "Company Name (or Web URL)"), 
    $orgUnit    = (Read-Host "Department (team, group, family)"), 
    $CommonName, 
    $email = (Read-Host "Email Address")
 )
 @"
 HOME			   = .
 RANDFILE		   = $($ENV::HOME)/.rnd
 
 
 ####################################################################
 [ req ]
 default_bits		   = {0}
 default_keyfile 	   = privkey.pem
 distinguished_name	= req_distinguished_name
 
 
 string_mask = nombstr
 
 [ req_distinguished_name ]
 countryName			      = Country Name (2 letter code)
 countryName_default		= {1}
 countryName_min			= 2
 countryName_max			= 2
 
 stateOrProvinceName		= State or Province Name (full name)
 stateOrProvinceName_default	= {2}
 
 localityName			= Locality Name (eg, city)
 localityName_default = {3}
 
 0.organizationName		= Organization Name (eg, company)
 0.organizationName_default	= {4}
 
 
 organizationalUnitName		      = Organizational Unit Name (eg, section)
 organizationalUnitName_default	= {5}
 
 commonName			= Common Name (eg, YOUR name)
 commonName_default = {6}
 commonName_max			= 64
 
 emailAddress			= Email Address
 emailAddress_default = {7}
 emailAddress_max		= 64
 
 
 
 [ v3_ca ]
 
 subjectKeyIdentifier=hash
 authorityKeyIdentifier=keyid:always,issuer:always
 
 
 basicConstraints = critical,CA:true
 
 
 
 [ code_sign ]
 subjectKeyIdentifier=hash
 authorityKeyIdentifier=keyid,issuer
 
 
 basicConstraints=CA:FALSE
 
 nsCertType = server, client, email, objsign
 extendedKeyUsage       = critical, serverAuth,clientAuth,codeSigning
 
 keyUsage = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
 
 nsComment			= "OpenSSL Generated Certificate"
 
 [ crl_ext ]
 
 authorityKeyIdentifier=keyid:always,issuer:always
 "@ -f $keyBits,$Country,$State,$city,$company,$orgUnit,$CommonName,$email
 }
 
 if(!$OpenSSLLocation) {
    $OpenSSLLocation = Split-Path $MyInvocation.MyCommand.Path
    Write-Debug "OpenSSL: $OpenSSLLocation"
 }
 if( Test-Path $OpenSslLocation ) {
    if($files.count -lt 3) {
       THROW "You need to configure a location where OpenSSL can be run from"
    }
 } else { THROW "You need to configure a location where OpenSSL can be run from" }
 
 [string]$SslCnfPath = (join-path (Convert-Path $CertStorageLocation) PoshOpenSSL.config)
 New-Alias OpenSsl (join-path $OpenSSLLocation OpenSSL.exe)
 
 if( !(Test-Path $CertStorageLocation) ) {
    New-Item -type directory -path $CertStorageLocation | Push-Location
    $forceNew = $true
 } else {
    Push-Location $CertStorageLocation
 }
 
 Write-Debug "SslCnfPath: $SslCnfPath"
 Write-Debug "OpenSsl: $((get-alias OpenSsl).Definition)"
 
 if($forceNew -or (@(Test-Path "$CodeSignName.crt","$CodeSignName.pfx") -contains $false)) {
 
    if($forceNew -or (@(Test-Path "$CodeSignName.key","$CodeSignName.csr") -contains $false)) {
 
       if($forceNew -or (@(Test-Path "$RootCAName.key","$RootCAName.crt") -contains $false)) {
 
          $CommonName = "$company Certificate Authority"
          $orgUnit = "$department Certificate Authority"
          $email = Get-UserEmail
 
          OpenSsl genrsa -out "$RootCAName.key" -des3 -passout pass:$(Get-RootCAPassword) $keyBits
          OpenSsl req -new -x509 -days $daysCA -key "$RootCAName.key" -out "$RootCAName.crt" -passin pass:$(Get-RootCAPassword) -config $SslCnfPath -batch
       }
 
       $CommonName = "$UserName"
       $orgUnit = "$department"
       $email = Get-UserEmail
 
       OpenSsl genrsa -out "$CodeSignName.key" -des3 -passout pass:$(Get-CodeSignPassword) $keyBits
       OpenSsl req -new -key "$CodeSignName.key" -out "$CodeSignName.csr" -passin pass:$(Get-CodeSignPassword) -config $SslCnfPath -batch
    }
 
    OpenSsl x509 -req -days $days -in "$CodeSignName.csr" -CA "$RootCAName.crt" -CAcreateserial -CAkey "$RootCAName.key" -out "$CodeSignName.crt" -setalias $alias -extfile $SslCnfPath -extensions code_sign -passin pass:$(Get-RootCAPassword)
    OpenSsl pkcs12 -export -out "$CodeSignName.pfx" -inkey "$CodeSignName.key" -in "$CodeSignName.crt" -passin pass:$(Get-CodeSignPassword) -passout pass:$script:CodeSignPassword
 }
 
 Pop-Location
 
 if($import) {
    
    trap {
       if($_.Exception.GetBaseException() -is [UnauthorizedAccessException]) {
          write-error "Cannot import certificates as 'Root CA' or 'Trusted Publisher' except in an elevated console."
          continue
       }
    }
    
    $lm = new-object System.Security.Cryptography.X509certificates.X509Store "root", "LocalMachine"
    $lm.Open("ReadWrite")
    $lm.Add( (Get-PfxCertificate "$CertStorageLocation\$RootCAName.crt") )
    if($?) {
       Write-Host "Successfully imported root certificate to trusted root store" -fore green
    }
    $lm.Close()
 
    $tp = new-object System.Security.Cryptography.X509certificates.X509Store "TrustedPublisher", "LocalMachine"
    $tp.Open("ReadWrite")
    $tp.Add( (Get-PfxCertificate "$CertStorageLocation\$CodeSignName.crt") )
    if($?) {
       Write-Host "Successfully imported code-signing certificate to trusted publishers store" -fore green
    }
    $tp.Close()
 
    if($importall) {
       $my = new-object System.Security.Cryptography.X509certificates.X509Store "My", "CurrentUser"
       $my.Open( "ReadWrite" )
       Get-CodeSignPassword
       $my.Add((Get-PfxCertificate "$CertStorageLocation\$CodeSignName.pfx"))      #$script:CodeSignPassword, $DefaultStorage)
       if($?) {
          Write-Host "Successfully imported code-signing certificate to 'my' store" -fore yellow
       }
       $my.Close()
    }
 }
 
`

