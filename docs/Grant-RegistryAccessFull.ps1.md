---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5489
Published Date: 2016-10-07t11
Archived Date: 2016-05-24t23
---

# grant-registryaccessfull - 

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
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Grants full control access to a user for the specified registry key.
 
 .EXAMPLE
 
 PS >$registryPath = "HKLM:\Software\MyProgram"
 PS >Grant-RegistryAccessFullControl "LEE-DESK\LEE" $registryPath
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $User,
 
     [Parameter(Mandatory = $true)]
     $RegistryPath
 )
 
 Set-StrictMode -Version Latest
 
 Push-Location
 Set-Location -LiteralPath $registryPath
 
 $acl = Get-Acl .
 
 $arguments = $user,"FullControl","Allow"
 $accessRule = New-Object Security.AccessControl.RegistryAccessRule $arguments
 $acl.SetAccessRule($accessRule)
 
 $acl | Set-Acl  .
 
 Pop-Location
`

