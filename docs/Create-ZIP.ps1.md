---
Author: joerg hochwald
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6434
Published Date: 2017-07-01t04
Archived Date: 2017-05-16t03
---

# create-zip - 

## Description

create a zip archive of a given file.

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
 
 
 function global:Create-ZIP {
 	<#
 			.SYNOPSIS
 			Create a ZIP archive of a given file
 
 			.DESCRIPTION
 			Create a ZIP archive of a given file.
 			By default within the same directory and the same name as the input
 			file.
 			This can be changed via command line parameters
 
 			.PARAMETER InputFile
 			Mandatory
 
 			The parameter InputFile is the file that should be compressed.
 			You can use it like this: "ClutterReport-20150617171648.csv",
 			or with a full path like this:
 			"C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv"
 
 			.PARAMETER OutputFile
 			Optional
 
 			You can use it like this: "ClutterReport-20150617171648",
 			or with a full path like this:
 			"C:\scripts\PowerShell\export\ClutterReport-20150617171648"
 
 			Do not append the extension!
 
 			.PARAMETER OutputPath
 			Optional
 
 			By default the new archive will be created in the same directory as the
 			input file, if you would like to have it in another directory specify
 			it here like this: "C:\temp\"
 
 			The directory must exist!
 
 			.EXAMPLE
 			PS C:\> Create-ZIP -InputFile "C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv"
 
 			Description
 			-----------
 			This will create the archive "ClutterReport-20150617171648.zip" from
 			the given input file
 			"C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv".
 
 			The new archive will be located in "C:\scripts\PowerShell\export\"!
 
 			.EXAMPLE
 			PS C:\> Create-ZIP -InputFile "C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv" -OutputFile "NewClutterReport"
 
 			Description
 			-----------
 			This will create the archive "NewClutterReport.zip" from the given
 			input file
 			"C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv".
 
 			The new archive will be located in "C:\scripts\PowerShell\export\"!
 
 			.EXAMPLE
 			PS C:\> Create-ZIP -InputFile "C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv" -OutputPath "C:\temp\"
 
 			Description
 			-----------
 			This will create the archive "ClutterReport-20150617171648.zip" from
 			the given input file
 			"C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv".
 
 			The new archive will be located in "C:\temp\"!
 
 			The directory must exist!
 
 			.EXAMPLE
 			PS C:\> Create-ZIP -InputFile "C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv" -OutputFile "NewClutterReport" -OutputPath "C:\temp\"
 
 			Description
 			-----------
 			This will create the archive "NewClutterReport.zip" from the given
 			input file
 			"C:\scripts\PowerShell\export\ClutterReport-20150617171648.csv".
 
 			The new archive will be located in "C:\temp\"!
 
 			The directory must exist!
 
 			.LINK
 			NET-Experts http://www.net-experts.net
 
 			.LINK
 			Support https://github.com/jhochwald/NETX/issues
 	#>
 
 	[CmdletBinding()]
 	param
 	(
 		[Parameter(Mandatory = $true,
 		HelpMessage = 'The parameter InputFile is the file that should be compressed (Mandatory)')]
 		[ValidateNotNullOrEmpty()]
 		[Alias('Input')]
 		[System.String]$InputFile,
 		[Alias('Output')]
 		[System.String]$OutputFile,
 		[System.String]$OutputPath
 	)
 
 	BEGIN {
 		Remove-Variable -Name MyFileName -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name MyFilePath -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name OutArchiv -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name zip -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 	}
 
 	PROCESS {
 		Set-Variable -Name MyFileName -Value $((Get-Item $InputFile).Name)
 
 		if (-not ($OutputFile)) {
 			Set-Variable -Name OutputFile -Value $((Get-Item $InputFile).BaseName)
 		}
 
 		Set-Variable -Name OutputFile -Value $($OutputFile + '.zip')
 
 		if (-not ($OutputPath)) {
 			Set-Variable -Name MyFilePath -Value $((Split-Path -Path $InputFile -Parent) + '\')
 		} else {
 			Set-Variable -Name OutputPath -Value $($OutputPath.TrimEnd('\'))
 
 			Set-Variable -Name MyFilePath -Value $(($OutputPath) + '\')
 		}
 
 		Set-Variable -Name OutArchiv -Value $(($MyFilePath) + ($OutputFile))
 
 		if (Test-Path $OutArchiv) {
 			Unblock-File -Path:$OutArchiv -Confirm:$false -ErrorAction:Ignore -WarningAction:Ignore
 
 			Remove-Item -Path:$OutArchiv -Force -Confirm:$false -ErrorAction:Ignore -WarningAction:Ignore
 		}
 
 		Add-Type -AssemblyName 'System.IO.Compression.FileSystem'
 
 		Set-Variable -Name zip -Value $([System.IO.Compression.ZipFile]::Open($OutArchiv, 'Create'))
 
 		$null = [System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile($zip, $InputFile, $MyFileName, 'optimal')
 
 		$zip.Dispose()
 
 		do {
 			Start-Sleep -Seconds:'1'
 		} while (($zip.Entries.count) -ne 0)
 
 		if ($RunUnattended) {
 			Write-Output -InputObject "$OutArchiv"
 		} else {
 			Write-Output -InputObject "Compressed: $InputFile"
 			Write-Output -InputObject "Archive: $OutArchiv"
 		}
 
 		Unblock-File -Path:$OutArchiv -Confirm:$false -ErrorAction:Ignore -WarningAction:Ignore
 	}
 
 	END {
 		Remove-Variable -Name MyFileName -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name MyFilePath -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name OutArchiv -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 		Remove-Variable -Name zip -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue
 	}
 }
 (Set-Alias -Name Create-Archive -Value Create-ZIP -Option:AllScope -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue) > $null 2>&1 3>&1
 (Set-Alias -Name Write-ZIP -Value Create-ZIP -Option:AllScope -Scope:Global -Force -Confirm:$false -ErrorAction:SilentlyContinue -WarningAction:SilentlyContinue) > $null 2>&1 3>&1
`

