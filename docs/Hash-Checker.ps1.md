---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 989
Published Date: 2009-04-01t13
Archived Date: 2015-08-16t00
---

# hash checker - 

## Description

check md5/sha1/etc hashes

## Comments



## Usage



## TODO



## script

`test-hash`

## Code

`#
 #
 #.Synopsis
 #.Description
 #.Example
 #
 #.Example
 #
 #
 #.Example
 #
 #
 function Test-Hash { 
 #[CmdletBinding(DefaultParameterSetName="NoExpectation")]
 PARAM(
    #[Parameter(Position=0,Mandatory=$true)]
    [string]$FileName
 ,
    #[Parameter(Position=2,Mandatory=$true,ParameterSetName="ManualHash")]
    [string]$ExpectedHash = $(if($HashFileName){  ((Get-Content $HashFileName) -match $FileName)[0].split(" ")[0]  })
 ,
    #[Parameter(Position=1,Mandatory=$true,ParameterSetName="FromHashFile")]
    [string]$HashFileName
 ,
    #[Parameter(Position=1,Mandatory=$true,ParameterSetName="ManualHash")]
    [string[]]$TypeOfHash = $(if($HashFileName){  
                           [IO.Path]::GetExtension((Convert-Path $HashFileName)).Substring(1) 
                  } else { "MD5","SHA1","SHA256","SHA384","SHA512","RIPEMD160" })
 )
 $ofs=""
   $hashes = @{}
   foreach($Type in $TypeOfHash) {
     [string]$hash = [Security.Cryptography.HashAlgorithm]::Create(
       $Type
     ).ComputeHash( 
       [IO.File]::ReadAllBytes( (Convert-Path $FileName) )
     ) | ForEach { "{0:x2}" -f $_ }
     $hashes.$Type = $hash
   }
   
   if($ExpectedHash) {
     ($hashes.Values -eq $hash).Count -ge 1
   } else {
     foreach($hash in $hashes.GetEnumerator()) {
       "{0,-8}{1}" -f $hash.Name, $hash.Value
     }        
   }
 }
`

