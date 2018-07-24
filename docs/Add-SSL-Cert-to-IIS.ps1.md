---
Author: brian h madsen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6634
Published Date: 2017-11-25t19
Archived Date: 2017-03-15t06
---

# add ssl cert to iis - 

## Description

how to add a ssl certificate to iis with powershell as well as set the ssl binding for the site that’s using the certificate.

## Comments



## Usage



## TODO



## function

`add-sslcertificate`

## Code

`#
 #
 function Add-SSLCertificate{
     param([string]$pfxPath,[string]$pfxPassword,[string]$hostHeader,[string]$siteName)
 
     $certMgr = New-Object -ComObject IIS.CertObj -ErrorAction SilentlyContinue    
     $certMgr.ImportToCertStore($pfxPath,$pfxPassword,$true,$true)
 
     Import-Module WebAdministration;
     New-WebBinding -Name $siteName -Port 443 -Protocol https -HostHeader $hostHeader    
 }
`

