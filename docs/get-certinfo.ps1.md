---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4955
Published Date: 
Archived Date: 2016-11-05t01
---

#  - 

## Description

powershell script to get certificate information for all the ad, for this you need ad user list in txt file and save them.

## Comments



## Usage



## TODO



## function

`get-certinfo`

## Code

`#
 #
 Function get-certinfo
 {
 $myobj = @()
 {
 $Cert = Get-ADUser $currentuser -Properties "Certificates"
 $tempobj = $cert.Certificates | foreach {New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 $_}
 
   if ($cert.Certificates.Handle -eq $null)
     {
     $rep = "" | select "Account", "Archived", "Extensions", "FriendlyName", "IssuerName", "NotAfter", "NotBefore", "HasPrivateKey", "PrivateKey", "PublicKey", "RawData", "SerialNumber", "SubjectName", "SignatureAlgorithm", "Thumbprint", "Version", "Handle", "Issuer", "Subject"
     $rep."Account" =  $currentuser
     $Rep."Archived" = "N/A"
     $Rep."Extensions" = "N/A"
     $Rep."FriendlyName" = "N/A"
     $Rep."IssuerName" = "N/A"
     $Rep."NotAfter" = "N/A"
     $Rep."NotBefore" = "N/A"
     $Rep."HasPrivateKey" = "N/A"
     $Rep."PrivateKey" = "N/A"
     $Rep."PublicKey" = "N/A"
     $Rep."RawData" = "N/A"
     $Rep."SerialNumber" = "N/A"
     $Rep."SubjectName" = "N/A"
     $Rep."SignatureAlgorithm" = "N/A"
     $Rep."Thumbprint" = "N/A"
     $Rep."Version" = "N/A"
     $Rep."Handle" = "N/A"
     $Rep."Issuer" = "N/A"
     $Rep."Subject" = "N/A"
     $MyObj += $Rep
     $rep = $null
     }
   else
     {
     $rep = "" | select "Account", "Archived", "Extensions", "FriendlyName", "IssuerName", "NotAfter", "NotBefore", "HasPrivateKey", "PrivateKey", "PublicKey", "RawData", "SerialNumber", "SubjectName", "SignatureAlgorithm", "Thumbprint", "Version", "Handle", "Issuer", "Subject"
     $rep."Account" =  $currentuser
     $Rep."Archived" = $tempobj."Archived"
     $Rep."Extensions" = $tempobj.Extensions
     $Rep."FriendlyName" = $tempobj.FriendlyName
     $Rep."IssuerName" = $tempobj.IssuerName
     $Rep."NotAfter" = $tempobj.NotAfter
     $Rep."NotBefore" = $tempobj.NotBefore
     $Rep."HasPrivateKey" = $tempobj.HasPrivateKey
     $Rep."PrivateKey" = $tempobj.PrivateKey
     $Rep."PublicKey" = $tempobj.PublicKey
     $Rep."RawData" = $tempobj.RawData
     $Rep."SerialNumber" = $tempobj.SerialNumber
     $Rep."SubjectName" = $tempobj.SubjectName
     $Rep."SignatureAlgorithm" = $tempobj.SignatureAlgorithm
     $Rep."Thumbprint" = $tempobj.Thumbprint
     $Rep."Version" = $tempobj.Version
     $Rep."Handle" = $tempobj.Handle
     $Rep."Issuer" = $tempobj.Issuer
     $Rep."Subject" = $tempobj.Subject
     $MyObj += $Rep
     $rep = $null
      }
 
 }
 }
`

