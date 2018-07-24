---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2334
Published Date: 
Archived Date: 2010-11-02t18
---

# get-objectidentifier - 

## Description

resolves oid value to a friendly name and vice versa. the cmdlet resolves both well-known oids (used in internet pki) and active directory forest specific registered oids.

## Comments



## Usage



## TODO



## function

`get-objectidentifier`

## Code

`#
 #
 #####################################################################
 #
 #
 #####################################################################
 
 function Get-ObjectIdentifier {
 <#
 .Synopsis
 	Resolves OID value to a Friendly Name and vice versa.
 .Description
 	Resolves OID value to a Friendly Name and vice versa. The cmdlet resolves both
 	well-known OIDs (used in Internet PKI) and Active Directory forest specific
 	registered OIDs.
 .Parameter OIDString
 	Specifies the OID value or Friendly name.
 .Example
 	Get-ObjectIdentifier "Server Authentication"
 	
 	Will resolve "Server Authentication" OID to an object identifier value (1.3.6.1.5.5.7.3.1).
 .Example
 	Get-ObjectIdentifier "1.3.6.1.5.5.7.3.9"
 	
 	Will resolve "1.3.6.1.5.5.7.3.9" value to a friendly name (OCSP Signing).
 .Outputs
 	System.Security.Cryptography.Oid
 #>
 [CmdletBinding()]
 [OutputType('System.Security.Cryptography.Oid')]
 	param (
 		[Parameter(Mandatory = $true, ValueFromPipeline = $true)]
 		[String[]]$OIDString
 	)
 	$OIDString | %{New-Object Security.Cryptography.Oid $OIDString}
 }
`

