---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5881
Published Date: 2016-06-02t20
Archived Date: 2016-09-06t03
---

# aes ctr mode decryptor - 

## Description

port of chris veness’ javascript aes ctr (aes counter mode) aes.ctr.decrypt to powershell. it will decrypt text encrypted with the aes.ctr.encrypt function from http

## Comments



## Usage



## TODO



## script

`convertfrom-aesctr`

## Code

`#
 #
 #.Synopsis
 #.Description
 #
 #
 #.Example
 #.Parameter Cypher
 #.Parameter Password
 #.Parameter Size
 #.Parameter InRaw
 #.Parameter OutRaw
 function ConvertFrom-AesCtr {
 	[OutputType([string], [System.ArraySegment[byte]])]
 	param(
 		[Parameter(Mandatory)]
 		$Cypher
 ,
 		[Parameter(Mandatory)]
 		[string]$Password
 ,
 		[ValidateSet(128, 192, 256)]
 		[int]$Size = 256
 ,
 		[switch]$InRaw
 ,
 		[switch]$OutRaw
 	)
 	try {
 		[int]$Size = $Size -shr 3
 
 		if ($InRaw) {
 			[byte[]]$Cypher = $Cypher
 		} else {
 			[byte[]]$Cypher = [System.Convert]::FromBase64String($Cypher)
 		}
 
 		[byte[]]$Password = [System.Text.Encoding]::UTF8.GetBytes($Password)
 		[array]::Resize([ref]$password, $Size)
 
 		$aes = [System.Security.Cryptography.Aes]::Create()
 		$aes.Mode = [System.Security.Cryptography.CipherMode]::ECB
 		$aes.Padding = [System.Security.Cryptography.PaddingMode]::Zeros
 		$aes.Key = $Password
 		$enc = $aes.CreateEncryptor()
 		[byte[]]$Password = $enc.TransformFinalBlock($Password, 0, $Password.Length)
 
 		[array]::Resize([ref]$Password, $Size)
 		[array]::Copy($Password, 0, $Password, 16, $Size - 16)
 
 		$aes.Key = $Password
 		$enc = $aes.CreateEncryptor()
 
 		$Password = New-Object byte[] 16
 		[array]::Copy($Cypher, $Password, 8)
 
 		foreach ($i in 0..[Math]::Floor(($Cypher.Length - 9) / 16)) {
 			$Password[15] = $i -band 255
 			$Password[14] = ($i -shr 8) -band 255
 			$Password[13] = ($i -shr 16) -band 255
 			$Password[12] = $i -shr 24
 
 			$aes = $enc.TransformFinalBlock($Password, 0, 16)
 			$i = $i * 16 + 8
 			foreach ($b in 0..[Math]::Min(15, $Cypher.Length - $i - 1)) {
 				$Cypher[$i+$b] = $Cypher[$i+$b] -bxor $aes[$b]
 			}
 		}
 
 		if ($OutRaw) {
 			,(New-Object System.ArraySegment[byte] @($Cypher, 8, ($Cypher.Length - 8)))
 		} else {
 			[System.Text.Encoding]::UTF8.GetString($Cypher, 8, $Cypher.Length - 8)
 		}
 	} catch {
 		throw
 	}
 }
`

