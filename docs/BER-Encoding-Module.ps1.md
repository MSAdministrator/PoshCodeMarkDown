---
Author: parul jain, paruljain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3379
Published Date: 2014-04-21t05
Archived Date: 2014-04-08t03
---

# ber encoding module - 

## Description

takes asn types integer, octet (string), octet (byte array), and oid (string) values and encodes into byte array using basic encoding rules (ber). ber encoded values are used for snmp, x.509 certificates, etc.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
     BER encoding library
 
 .DESCRIPTION
     Takes ASN types integer, octet (string), octet (byte array), and OID
     (string) values and encodes into byte array using Basic Encoding Rules (BER)
     BER encoding is used for SNMP, X.509 certificates, etc.
 
     There will be a companion BER decoding library posted soon. There will also
     be a SNMP library that makes use of BER encoding and decoding libraries
 
 .NOTES
     Author: Parul Jain, paruljain@hotmail.com
     Version: 0.1, April 21, 2012
     Requires: PowerShell v2 or better
 
 .EXAMPLE
     $varbindList = encOID '1.3.6.1.4.1.2680.1.2.7.3.2.0' | encNull | encSeq | encSeq
     $pdu = ((encInt 1 | encInt 0 | encInt 0) + $varbindList) | encSeq 0xA0
     $message = ((encInt 0 | encOctet 'private') + $pdu) | encSeq
 #>
 
 Add-Type @"
 public class Shift {
    public static long  Right(long x,  int count) { return x >> count; }
 }                    
 "@
 
 function byte2hex {
 
     [string]$ret = ''
     $input | % { $ret += '{0:X2} ' -f $_ }
     $ret.TrimEnd(' ')
 }
 
 function trimLeft([byte[]]$buffer) {
 
     $i = 0
     while ($buffer[$i] -eq 0) { $i++ } 
     $buffer[$i..($buffer.length-1)]
 }
 
 function encLength([long]$length) {
 
     if ($length -lt 128) { return [byte]$length }
     $buffer = [BitConverter]::GetBytes($length)
     [Array]::Reverse($buffer)
     [byte[]]$buffer = trimLeft $buffer
     @(128 + $buffer.length) + $buffer
 }
 
 function encInt([int]$value) {
 
     $b = [BitConverter]::GetBytes($value)
     [Array]::Reverse($b)
     [byte[]]$b = trimLeft $b
     $input + [byte[]](2, $b.length) + $b
 }
 
 function encOctet($buffer) {
 
     if ($buffer -is [string]) { $b = [Text.Encoding]::UTF8.GetBytes($buffer) }
         elseif ($buffer -is [byte[]]) { $b = $buffer }
             else { throw('Must be string or byte[] type') }
     $input + [byte[]](4, (encLength $b.length)) + $b
 }
 
 function encOID ([string]$oid) {
 
     $oid = ($oid.TrimStart('.')).TrimEnd('.')
     
     $octets = $oid.split('.')
     if ($octets.count -lt 2) { throw 'Error: Invalid OID; must have at least two octects' }
     if ([int]$octets[0] -gt 2 -or [int]$octets[1] -gt 39) { throw 'Error: Invalid OID' }
     [byte[]]$buffer = @()
     if ($octets.count -gt 2) {
         for($i=2; $i -lt $octets.count; $i++) {
             [byte[]]$buff= @()
             $value = [long]$octets[$i]
             do {
                 $b = [System.BitConverter]::GetBytes($value)
                 $b[0] = $b[0] -bor 0x80
                 $buff += $b[0]
                 $value = [shift]::right($value, 7)
             } until ($value -eq 0)
             $buff[0] = $buff[0] -band 0x7F
             [array]::Reverse($buff)
             $buffer += $buff
         }
     }
     $input + [byte[]](6, (encLength $buffer.length)) + $buffer
 }
 
 function encNull {
 
     $input + [byte[]](5, 0)
 }
 
 function encSeq([byte]$type=0x30) {
 
     $buffer = @($input)
     [byte[]]($type, (encLength $buffer.length)) + $buffer
 }
`

