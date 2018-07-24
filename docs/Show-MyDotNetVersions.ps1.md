---
Author: tzvika n
Publisher: 
Copyright: 
Email: 
Version: 4.5
Encoding: ascii
License: cc0
PoshCode ID: 6435
Published Date: 2016-07-01t04
Archived Date: 2016-07-04t03
---

# show-mydotnetversions - 

## Description

reads from the registry all the .net versions installed on the local machine.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 
 <#
 		#################################################
 		#################################################
 
 		Support: https://github.com/jhochwald/NETX/issues
 #>
 
 
 
 <#
 		Copyright (c) 2012-2016, NET-Experts <http:/www.net-experts.net>.
 		All rights reserved.
 
 		Redistribution and use in source and binary forms, with or without
 		modification, are permitted provided that the following conditions are met:
 
 		1. Redistributions of source code must retain the above copyright notice,
 		this list of conditions and the following disclaimer.
 
 		2. Redistributions in binary form must reproduce the above copyright notice,
 		this list of conditions and the following disclaimer in the documentation
 		and/or other materials provided with the distribution.
 
 		3. Neither the name of the copyright holder nor the names of its
 		contributors may be used to endorse or promote products derived from
 		this software without specific prior written permission.
 
 		THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 		AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 		IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 		ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 		LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 		CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 		SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 		INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 		CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 		ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 		THE POSSIBILITY OF SUCH DAMAGE.
 
 		By using the Software, you agree to the License, Terms and Conditions above!
 #>
 
 
 function global:Get-InstalledDotNetVersions {
 	<#
 			.SYNOPSIS
 			Shows all installed .Net versions
 
 			.DESCRIPTION
 			Shows all .Net versions installed on the local system
 
 			.EXAMPLE
 			PS C:\> Get-InstalledDotNetVersions
 
 			Version                                                                         FullVersion
 			-------                                                                         -----------
 			2.0                                                                             2.0.50727.5420
 			3.0                                                                             3.0.30729.5420
 			3.5                                                                             3.5.30729.5420
 			4.0                                                                             4.0.0.0
 			4.5+                                                                            4.6.1
 
 			Description
 			-----------
 			Shows all .Net versions installed on the local system
 
 			.NOTES
 			Based on Show-MyDotNetVersions from Tzvika N 9. I just tweaked the Code and removed the HTML parts
 			All Versions after .NET 4.5 will have the Version 4.5+ and show the full version in the FullVersion
 
 			.LINK
 			http://poshcode.org/6403
 	#>
 
 	[CmdletBinding()]
 	[OutputType([System.Object])]
 	param ()
 
 	BEGIN {
 		$RegistryBase = 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP'
 		$RegistryDotNet20 = "$($RegistryBase)\v2.0*"
 		$RegistryDotNet30 = "$($RegistryBase)\v3.0"
 		$RegistryDotNet35 = "$($RegistryBase)\v3.5"
 		$RegistryDotNet40 = "$($RegistryBase)\v4.0\Client"
 		$RegistryDotNet45 = "$($RegistryBase)\v4\Full"
 	}
 
 	PROCESS {
 		if (Test-Path $RegistryDotNet20) {
 			$DotNet20 = ((Get-ItemProperty -Path $RegistryDotNet20 -Name Version).Version)
 		}
 
 		if (Test-Path $RegistryDotNet30) {
 			$DotNet30 = ((Get-ItemProperty -Path $RegistryDotNet30 -Name Version).Version)
 		}
 
 		if (Test-Path $RegistryDotNet35) {
 			$DotNet35 = ((Get-ItemProperty -Path $RegistryDotNet35 -Name Version).Version)
 		}
 
 		if (Test-Path $RegistryDotNet40) {
 			$DotNet40 = ((Get-ItemProperty -Path $RegistryDotNet40 -Name Version).Version)
 		}
 
 		if (Test-Path $RegistryDotNet45) {
 			$verDWord = ((Get-ItemProperty -Path $RegistryDotNet45 -Name Release).Release)
 
 			switch ($verDWord) {
 				378389 { $DotNet45 = '4.5'
 				break }
 				378675 { $DotNet45 = '4.5.1'
 				break }
 				378758 { $DotNet45 = '4.5.1'
 				break }
 				379893 { $DotNet45 = '4.5.2'
 				break }
 				393295 { $DotNet45 = '4.6'
 				break }
 				393297 { $DotNet45 = '4.6'
 				break }
 				394254 { $DotNet45 = '4.6.1'
 				break }
 				394271 { $DotNet45 = '4.6.1'
 				break }
 				394747 { $DotNet45 = '4.6.2'
 				break }
 				394748 { $DotNet45 = '4.6.2'
 				break }
 				default { $DotNet45 = '4.5' }
 			}
 		}
 
 		$dotNetProperty20 = [ordered]@{
 			Version     = '2.0'
 			FullVersion = $DotNet20
 		}
 		$dotNetProperty30 = [ordered]@{
 			Version     = '3.0'
 			FullVersion = $DotNet30
 		}
 		$dotNetProperty35 = [ordered]@{
 			Version     = '3.5'
 			FullVersion = $DotNet35
 		}
 		$dotNetProperty40 = [ordered]@{
 			Version     = '4.0'
 			FullVersion = $DotNet40
 		}
 		$dotNetProperty45 = [ordered]@{
 			Version     = '4.5+'
 			FullVersion = $DotNet45
 		}
 
 		$dotNetObject20 = (New-Object -TypeName psobject -Property $dotNetProperty20)
 		$dotNetObject30 = (New-Object -TypeName psobject -Property $dotNetProperty30)
 		$dotNetObject35 = (New-Object -TypeName psobject -Property $dotNetProperty35)
 		$dotNetObject40 = (New-Object -TypeName psobject -Property $dotNetProperty40)
 		$dotNetObject45 = (New-Object -TypeName psobject -Property $dotNetProperty45)
 
 		$dotNetVersionObjects = $dotNetObject20, $dotNetObject30, $dotNetObject35, $dotNetObject40, $dotNetObject45
 	}
 
 	END {
 		Write-Output -InputObject $dotNetVersionObjects
 	}
 }
`

