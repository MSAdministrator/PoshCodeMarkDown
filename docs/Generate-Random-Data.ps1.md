---
Author: crazydave
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3638
Published Date: 2013-09-13t10
Archived Date: 2017-03-18t05
---

# generate random data - 

## Description

a few helper functions for generating some random data (in some specific formats/types)

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $dictionaryWords = gc $dictionaryFile
 $azLower = 'abcdefghijklmnopqrstuvwxyz'.ToCharArray()
 $azUpper = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.ToCharArray()
 $hex = '012345679ABCDEF'.ToCharArray()
 $digit = '0123456789'.ToCharArray()
 $alphanumeric = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'.ToCharArray()
 
 function RandomVIN 
 {
     $url = 'http://randomvin.com/getAjax.php?qry=random&str=random&p9=0'
     $client = New-Object System.Net.WebClient
     $client.DownloadString($url).SubString(4,17)
 }
 
 function RandomWord
 {
     $dictionaryWords | Get-Random
 }
 
 function RandomString([int] $len, [Array] $charSet = $azLower)
 {
     [String]::Join('', (1..$len | % { $charSet | Get-Random }))
 }
 
 function RandomPlate 
 {
     (RandomString -len 3 -charset $azUpper) + "-" + (Get-Random -min 100 -max 999)
 }
 
 function RandomIPv4 {
     [IPAddress]::Parse([String] (Get-Random) )
 }
 
 function RandomHexByte {
     "{0:X2}" -f (Get-Random -Minimum 0 -Maximum 255)
 }
 
 function RandomMAC {
     [String]::Join(":", (1..8 | % { RandomHexByte }))
 }
 
 function RandomDateTime {
     ([DateTime]::Now).AddYears((Get-Random -min -1 -max 1)).AddMonths((Get-Random -min -11 -max 11)).AddDays((Get-Random -min -30 -max 30)).AddHours((Get-Random -min -23 -max 23)).AddMinutes((Get-Random -min -59 -max 59)).AddSeconds((Get-Random -min -59 -max 59))
 }
`

