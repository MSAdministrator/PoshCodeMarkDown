---
Author: vadims podans
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1394
Published Date: 2009-10-14t00
Archived Date: 2015-11-12t23
---

# get-certificationauthori - 

## Description

retrieves all enterprise certification authorities (ca) in cuurent ad forest

## Comments



## Usage



## TODO



## function

`get-certificationauthority`

## Code

`#
 #
 #####################################################################
 #
 #
 ####################################################################
 
 function Get-CertificationAuthority ([string]$CAName) {
 	$domain = ([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).Name
 	$domain = "DC=" + $domain -replace '\.', ", DC="
 	$CA = [ADSI]"LDAP://CN=Enrollment Services, CN=Public Key Services, CN=Services, CN=Configuration, $domain"
 	$CAs = $CA.psBase.Children | %{
 		$current = "" | Select CAName, Computer
 		$current.CAName = $_ | %{$_.Name}
 		$current.Computer = $_ | %{$_.DNSHostName}
 		$current
 	}
 	if ($CAName) {$CAs = @($CAs | ?{$_.CAName -eq $CAName})}
 	if ($CAs.Count -eq 0) {throw "Sorry, here is no CA that match your search"}
 	$CAs
 }
`

