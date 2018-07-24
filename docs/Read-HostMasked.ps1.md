---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 104
Published Date: 2008-01-08t21
Archived Date: 2017-04-30t12
---

# read-hostmasked - 

## Description

read a string from the host using securestring input, but output it as a plain string for use in functions that don’t accept securestrings

## Comments



## Usage



## TODO



## function

`read-hostmasked`

## Code

`#
 #
 function Read-HostMasked([string]$prompt="Password") {
   $password = Read-Host -AsSecureString $prompt; 
   $BSTR = [System.Runtime.InteropServices.marshal]::SecureStringToBSTR($password);
   $password = [System.Runtime.InteropServices.marshal]::PtrToStringAuto($BSTR);
   [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR);
   return $password;
 }
`

