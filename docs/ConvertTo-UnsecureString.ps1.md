---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3461
Published Date: 
Archived Date: 2015-06-03t01
---

#  - 

## Description

powershell 3 script to call a jira rest service. does http basic pre-authentication (a plain invoke-restmethod does not work)

## Comments



## Usage



## TODO



## function

`convertto-unsecurestring`

## Code

`#
 #
 param($Issue, $Credentials = $(Get-Credential), $BaseURI = "https://your.jira.server/jira")
 
 function ConvertTo-UnsecureString(
     [System.Security.SecureString][parameter(mandatory=$true)]$SecurePassword)
 {
     $unmanagedString = [System.IntPtr]::Zero;
     try
     {
         $unmanagedString = [Runtime.InteropServices.Marshal]::SecureStringToGlobalAllocUnicode($SecurePassword)
         return [Runtime.InteropServices.Marshal]::PtrToStringUni($unmanagedString)
     }
     finally
     {
         [Runtime.InteropServices.Marshal]::ZeroFreeGlobalAllocUnicode($unmanagedString)
     }
 }
 
 function ConvertTo-Base64($string) {
    $bytes  = [System.Text.Encoding]::UTF8.GetBytes($string);
    $encoded = [System.Convert]::ToBase64String($bytes);
 
    return $encoded;
 }
 
 function ConvertFrom-Base64($string) {
    $bytes  = [System.Convert]::FromBase64String($string);
    $decoded = [System.Text.Encoding]::UTF8.GetString($bytes);
 
    return $decoded;
 }
 
 function Get-HttpBasicHeader($Credentials, $Headers = @{})
 {
 	$b64 = ConvertTo-Base64 "$($Credentials.UserName):$(ConvertTo-UnsecureString $Credentials.Password)"
 	$Headers["Authorization"] = "Basic $b64"
 	return $Headers
 }
 
 if($Issue) {
 	$uri = "$BaseURI/rest/api/2/issue/$Issue"
 } else {
 	$uri = "$BaseURI/rest/api/2/mypermissions" 
 }
 
 $headers = Get-HttpBasicHeader $Credentials
 Invoke-RestMethod -uri $uri -Headers $headers -ContentType "application/json"
`

