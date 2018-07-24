---
Author: jvarga
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3597
Published Date: 2012-08-28t04
Archived Date: 2012-09-05t18
---

# decrypt psi password - 

## Description

script courtesy of jaykul, i’m just reposting.  this script will decode the passwords for all accounts in the psi profile labeled “default”.

## Comments



## Usage



## TODO



## function

`decrypt-psi`

## Code

`#
 #
 function decrypt-psi ($jid, $pw) {
    $OFS = ""; $u = 0;
    for($p=0;$p -lt $pw.Length;$p+=4) {
       [char]([int]"0x$($pw[$p..$($p+3)])" -bxor [int]$jid[$u++])
    }
 }
 
 $accounts = ([xml](cat ~\psidata\profiles\default\accounts.xml))["accounts"]["accounts"]
 
 foreach($account in ($accounts | gm a[0-9]*)) {
    $a = $accounts.$($account.Name) 
 }
`

