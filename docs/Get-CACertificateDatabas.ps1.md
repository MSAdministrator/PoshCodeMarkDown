---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5793
Published Date: 2017-03-25t15
Archived Date: 2017-05-13t16
---

# get-cacertificatedatabas - 

## Description

function for retrieving certificates from a ca instance. it can also return the public key, which i use to encrypt credentials in dsc resources (thumbprint is also returned).

## Comments



## Usage



## TODO



## function

`get-cacertificatedatabase`

## Code

`#
 #
 function Get-CACertificateDatabase
 {
     <#
     .SYNOPSIS
     Retrieves information about certificates from the Certificate Authority Database
 
     .DESCRIPTION
     This function will fetch items from a Certificate Authority Database. It can also 
     fetch the public key of the certificates and the thumbprint which could be really
     useful when you want to use the certificates to for example encrypt something
     (like a credential in a DSC resource).
 
     Another useful scenario is to create monitoring of certificate expiration dates.
 
     .EXAMPLE
     Get-CACertificateDatabase -CA "myca.contoso.com\Issuing CA Contoso" -IncludeBinaryCertificate
 
     Fetch certificates from the CA instance and include the public key.
 
     .EXAMPLE
     Get-CACertificateDatabase -CA "myca.contoso.com\Issuing CA Contoso" -ValidTo (Get-Date)
 
     Fetch certificates that expires today.
 
     .PARAMETER CertificationAuthority
     The Certificate Authority instance you want to connect to. For example:
     'myca.contoso.com\Issuing CA Contoso'
 
     .PARAMETER ValidFrom
     Filter what certificates should be returned based on if they are valid at this date.
 
     .PARAMETER ValidTo
     Filter what certificates should be returned based on if they expire before this date.
 
     .PARAMETER Disposition
     Specifies which category to get the certificates from.
 
     Brief disposition code explanation:
     * 9 - pending for approval
     * 15 - CA certificate renewal
     * 16 - CA certificate chain
     * 20 - issued certificates
     * 21 - revoked certificates
     * all other - failed requests
 
     .PARAMETER IncludeBinaryCertificate
     This switch will enable retrieval of the public key of the certificates.
 
     #>
 
     [cmdletbinding()]
     param ([parameter(Mandatory = $true)]
            [string] $CertificationAuthority,
            [parameter(Mandatory = $false)]
            [datetime] $ValidFrom = (Get-Date),
            [parameter(Mandatory = $false)]
            [datetime] $ValidTo = (Get-Date).AddYears(2),
            [parameter(Mandatory = $false)]
            [int] $Disposition = 20,
            [parameter(Mandatory = $false)]
            [switch] $IncludeBinaryCertificate)
 
     BEGIN { }
 
     PROCESS {
 
         Write-Verbose 'Initiating com object'
 
         $CaView = New-Object -Com CertificateAuthority.View
 
         try {
             Write-Verbose "Connecting to $CertificationAuthority..."
             [void] $CaView.OpenConnection($CertificationAuthority)
         }
         catch {
             Write-Error "Failed to connect to the Certificate Authority instance $CA. The error was: $($_.toString())"
             break
         }
 
         $CaView.SetResultColumnCount(8)
 
         $index0 = $CaView.GetColumnIndex($false, "Issued Common Name")
         $index1 = $CaView.GetColumnIndex($false, "Certificate Expiration Date")
         $index2 = $CaView.GetColumnIndex($false, "Issued Email Address")
         $index3 = $CaView.GetColumnIndex($false, "Certificate Template")
         $index4 = $CaView.GetColumnIndex($false, "Request Disposition")
 
         if ($IncludeBinaryCertificate) {
             $index5 = $CaView.GetColumnIndex($false, "Binary Certificate")
         }
 
         $index6 = $CaView.GetColumnIndex($false, "Certificate Hash")
         $index7 = $CaView.GetColumnIndex($false, "Requester Name")
 
         $index0, $index1, $index2, $index3, $index4, $index5, $index6, $index7 | ForEach-Object { $CAView.SetResultColumn($_) }
 
 
 
         $index1 = $CaView.GetColumnIndex($false, "Certificate Expiration Date")
         $CAView.SetRestriction($index1,16,0,$ValidFrom)
         $CAView.SetRestriction($index1,2,0,$ValidTo)
 
 
         $CAView.SetRestriction($index4,1,0,$Disposition)
 
         $RowObj= $CAView.OpenView()
 
         try {
             Write-Verbose 'Fetching certificates...'
 
             while ($Rowobj.Next() -ne -1) {
                 $Cert = New-Object PsObject
                 $ColObj = $RowObj.EnumCertViewColumn()
                 [void]$ColObj.Next()
 
                 do {
                     $current = $ColObj.GetName()
                     if ($ColObj.GetDisplayName() -eq 'Certificate Hash') {
                         $Cert | Add-Member -MemberType NoteProperty 'Thumbprint' -Value $($ColObj.GetValue(1).ToUpper() -replace "\s") -Force
                     }
                     elseif ($ColObj.GetDisplayName() -eq 'Binary Certificate') {
                         $Cert | Add-Member -MemberType NoteProperty 'BinaryCertificate' -Value "-----BEGIN CERTIFICATE-----`n$($ColObj.GetValue(1))-----END CERTIFICATE-----" -Force
                     }
                     else {
                         $Cert | Add-Member -MemberType NoteProperty $($ColObj.GetDisplayName() -replace '\s') -Value $($ColObj.GetValue(1)) -Force
                     }
 
                 } until ($ColObj.Next() -eq -1)
 
                 Clear-Variable ColObj
 
                 Write-Output $Cert
             }
         }
         catch {
             if ($_.toString() -like '*CEnumCERTVIEWROW::Next: The parameter is incorrect. 0x80070057*') {
                 Write-Verbose "No certificates matched the criteria in the database of $CertificationAuthority"
             }
             else {
                 Write-Error $_.toString()
             }
         }
     }
 
     END {
 
         Write-Verbose 'Cleaning up...'
 
         $RowObj.Reset()
         $CaView = $null
         [GC]::Collect()
 
         Write-Verbose 'Function finished.'
     }
 }
`

