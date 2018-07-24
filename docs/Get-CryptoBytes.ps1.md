---
Author: webclient
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6655
Published Date: 2016-12-19t16
Archived Date: 2016-12-30t16
---

# get-cryptobytes - 

## Description

generate cryptographically random bytes, using rngcryptoserviceprovider, and optionally format them as strings.

## Comments



## Usage



## TODO



## function

`get-cryptobytes`

## Code

`#
 #
 function Get-CryptoBytes {
 #.Synopsis
 #.Description
 #.Parameter Count
 #.Parameter AsString
 param(
    [Parameter(ValueFromPipeline=$true)]
    [int[]]$count = 64
 ,
    [switch]$AsString
 )
 
 begin {
    $RNGCrypto = New-Object System.Security.Cryptography.RNGCryptoServiceProvider
    $OFS = ""
 }
 process {
    foreach($length in $count) {
       $bytes = New-Object Byte[] $length
       $RNGCrypto.GetBytes($bytes)
       if($AsString){
          Write-Output ("{0:X2}" -f $bytes)
       } else {
          Write-Output $bytes
       }
    }
 }
 end {
    $RNGCrypto = $null
 }
 }
`

