---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1768
Published Date: 
Archived Date: 2010-04-17t07
---

#  - 

## Description

quick script signer using the last avalable codesigning cert in my cert store

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $cert1 = get-childitem  cert:\currentuser\my -CodeSigningCert | 
     ?{$_.Subject -eq "E=CoE@contoso.com"} | 
      sort-object NotBefore |select-object -last 1
 
 $r= Set-AuthenticodeSignature c:\signing\Script2.ps1 -Cert $cert1 -Force -Verbose  -IncludeChain "All" -TimeStampServer "http://timestamp.verisign.com/scripts/timstamp.dll"
 $r | FL
 
 
 Get-AuthenticodeSignature C:\Signing\Script2.ps1 | fl
 
 Sign 
 
 
 
       if($CertPath) {
          Set-AuthenticodeSignature -FilePath $file -Certificate $CertPath
       } else {
          Set-AuthenticodeSignature -FilePath $file
       }
 
 
 
 Validate Signature of scripts (List non Valid script ) 
 
 $Folder 
 
 ForEach($file in Get-ChildItem $Folder | Get-AuthenticodeSignature | 
       Where-Object { $_.Status -ne "Valid" -and $_.StatusMessage -ne $invalidForm } | 
       Select-Object -ExpandProperty Path ) 
 {
 	
 }
 
 
 dir ?.ps1 | Get-AuthenticodeSignature | % {if($_.TimeStamperCertificate -eq $null){write-warning "no time stamp"
 };$_}
`

