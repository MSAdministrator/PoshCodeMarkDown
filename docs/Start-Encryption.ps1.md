---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4487
Published Date: 2014-09-25t18
Archived Date: 2017-02-15t18
---

# start-encryption - 

## Description

functions to encrypt and decrypt strings using the rijndael symmetric key algorithm

## Comments



## Usage



## TODO



## function

`encrypt-string`

## Code

`#
 #
 ##################################################################################################
 ##
 
 [Reflection.Assembly]::LoadWithPartialName("System.Security")
 
 function Encrypt-String($String, $Passphrase, $salt="My Voice is my P455W0RD!", $init="Yet another key", [switch]$arrayOutput)
 {
    $r = new-Object System.Security.Cryptography.RijndaelManaged
    $pass = [Text.Encoding]::UTF8.GetBytes($Passphrase)
    $salt = [Text.Encoding]::UTF8.GetBytes($salt)
 
    $r.IV = (new-Object Security.Cryptography.SHA1Managed).ComputeHash( [Text.Encoding]::UTF8.GetBytes($init) )[0..15]
    
    $c = $r.CreateEncryptor()
    $ms = new-Object IO.MemoryStream
    $cs = new-Object Security.Cryptography.CryptoStream $ms,$c,"Write"
    $sw = new-Object IO.StreamWriter $cs
    $sw.Write($String)
    $sw.Close()
    $cs.Close()
    $ms.Close()
    $r.Clear()
    [byte[]]$result = $ms.ToArray()
    if($arrayOutput) {
       return $result
    } else {
       return [Convert]::ToBase64String($result)
    }
 }
 
 function Decrypt-String($Encrypted, $Passphrase, $salt="My Voice is my P455W0RD!", $init="Yet another key")
 {
    if($Encrypted -is [string]){
       $Encrypted = [Convert]::FromBase64String($Encrypted)
    }
 
    $r = new-Object System.Security.Cryptography.RijndaelManaged
    $pass = [System.Text.Encoding]::UTF8.GetBytes($Passphrase)
    $salt = [System.Text.Encoding]::UTF8.GetBytes($salt)
 
    $r.IV = (new-Object Security.Cryptography.SHA1Managed).ComputeHash( [Text.Encoding]::UTF8.GetBytes($init) )[0..15]
 
    $d = $r.CreateDecryptor()
    $ms = new-Object IO.MemoryStream @(,$Encrypted)
    $cs = new-Object Security.Cryptography.CryptoStream $ms,$d,"Read"
    $sr = new-Object IO.StreamReader $cs
    Write-Output $sr.ReadToEnd()
    $sr.Close()
    $cs.Close()
    $ms.Close()
    $r.Clear()
 }
`

