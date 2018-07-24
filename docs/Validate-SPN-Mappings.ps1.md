---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1613
Published Date: 
Archived Date: 2010-12-30t15
---

# validate spn mappings - spnvalidation.psm1

## Description

this is a script module with two functions

## Comments



## Usage



## TODO



## script

`resolve-spn`

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 function Resolve-SPN {
 ################################################################
 #.Synopsis
 #.Parameter SPN
 ################################################################
 param( [Parameter(Mandatory=$true)][string]$SPN)
 
 $Filter = "(ServicePrincipalName=$SPN)"
 $Searcher = New-Object System.DirectoryServices.DirectorySearcher
 $Searcher.Filter = $Filter
 $SPNEntry = $Searcher.FindAll()
 $Count = $SPNEntry | Measure-Object
 
 if ($Count.Count -gt 1) {
 Write-Host "Duplicate SPN Found!" -ForegroundColor Red -BackgroundColor Black
 Write-Host "The following Active Directory objects contains the SPN $SPN :"
 $SPNEntry | Format-Table Path -HideTableHeaders
 }
 
 elseif ($Count.Count -eq 1) {
 Write-Host "No duplicate SPN found" -ForegroundColor Green
 Write-Host "The following Active Directory objects contains the SPN $SPN :"
 $SPNEntry | Format-Table Path -HideTableHeaders
 }
 
 else
 
 {
 Write-Host "SPN not found"
 }
 }
 
 Import-Module ActiveDirectory
 
 function Resolve-AllDuplicateDomainSPNs {
 ################################################################
 #.Synopsis
 ################################################################
 
 $DomainSPNs = Get-ADObject -LDAPFilter "(ServicePrincipalName=*)" -Properties ServicePrincipalName
 
 foreach ($item in $DomainSPNs) {
 $SPNCollection = $item.ServicePrincipalName
 
 foreach ($SPN in $SPNCollection){
 $Filter = "(ServicePrincipalName=$SPN)"
 $Searcher = New-Object System.DirectoryServices.DirectorySearcher
 $Searcher.Filter = $Filter
 $SPNEntry = $Searcher.FindAll()
 $Count = $SPNEntry | Measure-Object
 
 if ($count.Count -gt 1) {
 Write-Host "Warning: Duplicate SPN Found!" -ForegroundColor Red -BackgroundColor Black
 Write-Host "The following Active Directory objects contains the SPN $SPN :"
 $SPNEntry | Format-Table Path -HideTableHeaders
  }
  }
  }
 }
`

