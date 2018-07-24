---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4420
Published Date: 2014-08-26t18
Archived Date: 2017-05-26t18
---

# import-pfxcertificate - 

## Description

import certificate files to the cert store

## Comments



## Usage



## TODO



## function

`import-pfxcertificate`

## Code

`#
 #
 function Import-PfxCertificate {
     #.Synopsis
     #.Example
     #
     #.Example
     #
     [CmdletBinding()]
     param( 
         [Parameter(ValueFromPipelineByPropertyName=$true, Position=0)]
         [Alias("PSPath")]
         [String]$PfxCertificatePath = "*.pfx", 
         [Parameter(ValueFromPipelineByPropertyName=$true, Position=1)]
         [Alias("Target","Store")]
         [String]$CertificateStorePath = "Cert:\CurrentUser\My"
     )
 
     process {
         $store = Get-Item $CertificateStorePath -EA 0 -EV StoreError | Where { $_ -is [System.Security.Cryptography.X509Certificates.X509Store] }
         if(!$Store) {
             $store = Get-Item Cert:\$CertificateStorePath -EA 0| Where { $_ -is [System.Security.Cryptography.X509Certificates.X509Store] }
             if(!$Store) { throw "Couldn't find X509 Certificate Store: $StoreError" }
             $CertificateStorePath = "Cert:\$CertificateStorePath"
         }
 
         try {
             $store.Open("MaxAllowed")
         } catch {
             throw "Couldn't open x509 Certificate Store: $_"
         }
 
         foreach($certFile in Get-Item $PfxCertificatePath) {
             Write-Warning "Attempting to load $($certfile.Name)"
             $cert = Get-PfxCertificate -LiteralPath $certFile.FullName
             if(!$cert) {
                 Write-Warning "Failed to load $($certfile.Name)"
                 continue
             }
             $store.Add($cert)
 
             Get-Item "${CertificateStorePath}$($cert.Thumbprint)"
         }
         $store.Close()
     }
 }
`

