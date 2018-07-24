---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5815
Published Date: 2016-04-06t14
Archived Date: 2016-05-24t22
---

# get-filehash.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

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
 
 Get the hash of an input file.
 
 .EXAMPLE
 
 Get-FileHash myFile.txt
 Gets the hash of a specific file
 
 .EXAMPLE
 
 dir | Get-FileHash
 Gets the hash of files from the pipeline
 
 .EXAMPLE
 
 Get-FileHash myFile.txt -Hash SHA1
 Gets the has of myFile.txt, using the SHA1 hashing algorithm
 
 #>
 
 param(
     $Path,
 
     [ValidateSet("MD5", "SHA1", "SHA256", "SHA384", "SHA512")]
     $HashAlgorithm = "MD5"
 )
 
 Set-StrictMode -Version Latest
 
 $hashType = [Type] "System.Security.Cryptography.$HashAlgorithm"
 $hasher = $hashType::Create()
 
 $files = @()
 
 if($path)
 {
     $files += $path
 }
 else
 {
     $files += @($input | Foreach-Object { $_.FullName })
 }
 
 foreach($file in $files)
 {
     if(-not (Test-Path $file -Type Leaf)) { continue }
 
     $filename = (Resolve-Path $file).Path
 
     $inputStream = New-Object IO.StreamReader $filename
     $hashBytes = $hasher.ComputeHash($inputStream.BaseStream)
     $inputStream.Close()
 
     $builder = New-Object System.Text.StringBuilder
     $hashBytes | Foreach-Object { [void] $builder.Append($_.ToString("X2")) }
 
     $output = New-Object PsObject -Property @{
         Path = ([IO.Path]::GetFileName($file));
         HashAlgorithm = $hashAlgorithm;
         HashValue = $builder.ToString()
     }
 
     $output
 }
`

