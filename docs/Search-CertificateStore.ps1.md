---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 6072
Published Date: 2016-10-29t15
Archived Date: 2016-05-24t22
---

# search-certificatestore. - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Search the certificate provider for certificates that match the specified
 Enhanced Key Usage (EKU.)
 
 .EXAMPLE
 
 Search-CertificateStore "Encrypting File System"
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $EkuName
 )
 
 $cert = Get-ChildItem cert:\CurrentUser\My
 
 Set-StrictMode -Off
 
 foreach($cert in $cert)
 {
     foreach($extension in $cert.Extensions)
     {
         foreach($certEku in $extension.EnhancedKeyUsages)
         {
             if($certEku.FriendlyName -eq $EkuName)
             {
                 $cert
             }
         }
     }
 }
`

