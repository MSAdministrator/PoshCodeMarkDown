---
Author: akaneo
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5683
Published Date: 2015-01-12t10
Archived Date: 2015-01-31t20
---

# get-windowsproduct - 

## Description

this is the “proper” version of get-windowsproductkey

## Comments



## Usage



## TODO



## function

`get-windowsproduct`

## Code

`#
 #
 Function Get-WindowsProduct
 {
 	function DecodeDigitalPID([byte[]]$digitalProductId)
 	{
 		$base24 = 'BCDFGHJKMPQRTVWXY2346789'
 		$cryptedStringLength = 24
 		$decryptionLength = 14
 		$decryptedKey = [System.String]::Empty
 
 		[System.Boolean]$containsN = ($digitalProductId[$decryptionLength] -shr 3) -band 1
 		[System.Byte]$digitalProductId[$decryptionLength] = $digitalProductId[$decryptionLength] -band 0xF7
 
 		for ($i = $cryptedStringLength; $i -ge 0; $i--)
 		{
 			$digitMapIndex = 0
 			for ($j = $decryptionLength; $j -ge 0; $j--)
 			{
 				[System.UInt16]$digitMapIndex = $digitMapIndex -shl 8 -bxor $digitalProductId[$j]
 				[System.Byte]$digitalProductId[$j] = [System.Math]::Floor($digitMapIndex / $base24.Length)
 				[System.UInt16]$digitMapIndex = $digitMapIndex % $base24.Length
 			}
 			$decryptedKey = $decryptedKey.Insert(0, $base24[$digitMapIndex])
 		}
 
 		if ($containsN)
 		{
 			$firstCharIndex = 0
 			for ($index = 0; $index -lt $cryptedStringLength; $index++)
 			{
 				if ($decryptedKey[0] -ne $base24[$index]) { continue }
 				$firstCharIndex = $index; break
 			}
 			$decryptedKey = $decryptedKey.Remove(0, 1).Insert($firstCharIndex,'N')
 		}
 
 		for ($t = 20; $t -ge 5; $t -= 5) { $decryptedKey = $decryptedKey.Insert($t, '-') }
 
 		return $decryptedKey
 	}
 
 	$regKey = Get-ItemProperty -Path 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion'
 
 	return [PSCustomObject]@{
 		Version		= [System.Environment]::OSVersion.VersionString
 		Edition		= $regKey.EditionID
 		Type		= switch ([System.Environment]::Is64BitOperatingSystem) { $true {'x86-64'}; $false {'x86-32'}}
 		ProductID	= $regKey.ProductId
 		ProductKey	= DecodeDigitalPID($regKey.DigitalProductId[0x34..0x42])
 		Installed	= [System.DateTime]::FromBinary(0x089F7FF5F7B58000).AddSeconds($regKey.InstallDate).ToLocalTime()
 	}
 }
`

