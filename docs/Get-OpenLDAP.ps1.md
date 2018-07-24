---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 86
Published Date: 2008-12-31t14
Archived Date: 2016-10-31t11
---

# get-openldap.ps1 - 

## Description

this script performs openldap query against specified server.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 Param($user,
       $password = $(Read-Host "Enter Password" -asSec),
       $filter = "(objectclass=user)",
       $server = $(throw '$server is required'),
       $path = $(throw '$path is required'),
       [switch]$all,
       [switch]$verbose)
     
 function GetSecurePass ($SecurePassword) {
   $Ptr = [System.Runtime.InteropServices.Marshal]::SecureStringToCoTaskMemUnicode($SecurePassword)
   $password = [System.Runtime.InteropServices.Marshal]::PtrToStringUni($Ptr)
   [System.Runtime.InteropServices.Marshal]::ZeroFreeCoTaskMemUnicode($Ptr)
   $password
 }    
 
 if($verbose){$verbosepreference = "Continue"}
 
 $DN = "LDAP://$server/$path"
 Write-Verbose "DN = $DN"
 $auth = [System.DirectoryServices.AuthenticationTypes]::FastBind
 Write-Verbose "Auth = FastBind"
 $de = New-Object System.DirectoryServices.DirectoryEntry($DN,$user,(GetSecurePass $Password),$auth)
 Write-Verbose $de
 Write-Verbose "Filter: $filter"
 $ds = New-Object system.DirectoryServices.DirectorySearcher($de,$filter) 
 Write-Verbose $ds
 if($all)
 {
     Write-Verbose "Finding All"
     $ds.FindAll()
 }
 else
 {
     Write-Verbose "Finding One"
     $ds.FindOne()
 }
`
