---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 5234
Published Date: 2016-06-11t04
Archived Date: 2016-05-19t12
---

# get-charinfo - 

## Description

gets unicode information about a character. if you have charmap.exe installed it will use its getuname.dll to get a descriptive string about the character.

## Comments



## Usage



## TODO



## function

`get-charinfo`

## Code

`#
 #
 <#
 
 Use this like this:
 
 "abc",([char]'x'),0xBF | Get-CharInfo
 
 #>
 
 Set-StrictMode -Version latest
 
 Add-Type -Name UName -Namespace Microsofts.CharMap -MemberDefinition $(
 	switch ("$([System.Environment]::SystemDirectory -replace '\\','\\')\\getuname.dll") {
 	{Test-Path -LiteralPath $_ -PathType Leaf} {@"
 [DllImport("${_}", ExactSpelling=true, SetLastError=true)]
 private static extern int GetUName(ushort wCharCode, [MarshalAs(UnmanagedType.LPWStr)] System.Text.StringBuilder buf);
 
 public static string Get(char ch) {
 	var sb = new System.Text.StringBuilder(300);
 	UName.GetUName(ch, sb);
 	return sb.ToString();
 }
 "@
 	}
 	default {'public static string Get(char ch) { return "???"; }'}
 	})
 
 function Get-CharInfo {
 	[CmdletBinding()]
 	param(
 		[Parameter(Position = 0, Mandatory, ValueFromPipeline)]
 		$InputObject
 	)
 	begin {
 		function out {
 			param($ch)
 			if (0 -le $ch -and 0xFFFF -ge $ch) {
 				[pscustomobject]@{
 					Char = [char]$ch
 					CodePoint = 'U+{0:X4}' -f $ch
 					Category = [System.Globalization.CharUnicodeInfo]::GetUnicodeCategory($ch)
 					Description = [Microsofts.CharMap.UName]::Get($ch)
 				}
 			} elseif (0 -le $ch -and 0x10FFFF -ge $ch) {
 				$s = [char]::ConvertFromUtf32($ch)
 				[pscustomobject]@{
 					Char = $s
 					CodePoint = 'U+{0:X}' -f $ch
 					Category = [System.Globalization.CharUnicodeInfo]::GetUnicodeCategory($s, 0)
 					Description = '???'
 				}
 			} else {
 				Write-Warning ('Character U+{0:X} is out of range' -f $ch)
 			}
 		}
 	}
 	process {
 		if ($null -cne ($InputObject -as [char])) {
 			out ([int][char]$InputObject)
 		} elseif ($InputObject -isnot [string] -and $null -cne ($InputObject -as [int])) {
 			out ([int]$InputObject)
 		} else {
 			$InputObject = [string]$InputObject
 			for ($i = 0; $i -lt $InputObject.Length; ++$i) {
 				if ([char]::IsHighSurrogate($InputObject[$i]) -and (1+$i) -lt $InputObject.Length -and [char]::IsLowSurrogate($InputObject[$i+1])) {
 					out ([char]::ConvertToUtf32($InputObject, $i))
 					++$i
 				} else {
 					out ([int][char]$InputObject[$i])
 				}
 			}
 		}
 	}
 }
`

