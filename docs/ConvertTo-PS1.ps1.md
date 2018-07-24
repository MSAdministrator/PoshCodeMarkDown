---
Author: mario
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: utf-8
License: mitl
PoshCode ID: 6831
Published Date: 2017-04-04t18
Archived Date: 2017-05-16t03
---

# convertto-ps1 - 

## Description

transform nested hashtable into powershell-literal (string)

## Comments



## Usage



## TODO



## functions

`convertto-ps1`

## Code

`#
 #
 #
 '@
 
 
 #-- Transform nested hashtable into Powershell-literal (string)
 function ConvertTo-PS1() {
     param($hash, $indent="", $depth=100, $CRLF="`r`n", $Q="'", $v2=@{})
     $sub = $indent + "    "
     switch ($hash.GetType().Name) {
         Int32 {}
         Double {}
         PSCustomObject {
             $hash.PSObject.Properties | ? { $_.Name } | % { $v2[$_.Name] = $_.Value }
             $hash = ConvertTo-PS1 $v2 $indent
         }
         Hashtable {
             $hash = "@{$CRLF" + (($hash.keys | % {
                 $sub + (ConvertTo-PS1 $_ $sub) + " = " + (ConvertTo-PS1 $hash[$_] $sub)
             }) -join ";$CRLF") + "$CRLF$indent}"
         }
         default { $hash = "$Q$value$Q"; }
     }
     return "$hash"
 }
`

