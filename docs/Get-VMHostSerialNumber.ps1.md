---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1341
Published Date: 2010-09-23t10
Archived Date: 2015-01-26t04
---

# get-vmhostserialnumber - 

## Description

get esx server serial numbers.

## Comments



## Usage



## TODO



## function

`get-vmhostserialnumber`

## Code

`#
 #
 function Get-VMHostSerialNumber {
 	param([VMware.VimAutomation.Types.VMHost[]]$InputObject = $null)
 
 	process {
 		$hView = $_ | Get-View -Property Hardware
 		$serviceTag =  $hView.Hardware.SystemInfo.OtherIdentifyingInfo | where {$_.IdentifierType.Key -eq "ServiceTag" }
 		$assetTag =  $hView.Hardware.SystemInfo.OtherIdentifyingInfo | where {$_.IdentifierType.Key -eq "AssetTag" }
 		$obj = New-Object psobject
 		$obj | Add-Member -MemberType NoteProperty -Name VMHost -Value $_
 		$obj | Add-Member -MemberType NoteProperty -Name ServiceTag -Value $serviceTag.IdentifierValue
 		$obj | Add-Member -MemberType NoteProperty -Name AssetTag -Value $assetTag.IdentifierValue
 		Write-Output $obj
 	}
 }
`

