---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3912
Published Date: 2013-01-23t18
Archived Date: 2013-01-31t03
---

# localstorage module - 

## Description

a quick and simple module for storing stuff in appdata\local

## Comments



## Usage



## TODO



## script

`get-localstoragepath`

## Code

`#
 #
 if($Args) { 
 	[string]$script:LocalStorageModuleName = $Args[0] 
 } elseif($LocalStorageModuleName) { 
 	[string]$script:LocalStorageModuleName = $LocalStorageModuleName
 } else {
 	[string]$script:LocalStorageModuleName = "LocalStorage" 
 }
 
 function Get-LocalStoragePath {
 	#.Synopsis
 	#.Description
 	param(
 		[Parameter(Position=0)]
 		[ValidateScript({ 
 			$invalid = $_.IndexOfAny([IO.Path]::GetInvalidFileNameChars())			
 			if($invalid -eq -1){ 
 				return $true
 			} else {
 				throw "Invalid character in Module Name '$_' at $invalid"
 			}
 		})]			
 		[string]$Module = $LocalStorageModuleName,
 
 		[Parameter(Position=1)]
 		[ValidateScript({ 
 			$invalid = $_.IndexOfAny([IO.Path]::GetInvalidFileNameChars())			
 			if($invalid -eq -1){ 
 				return $true
 			} else {
 				throw "Invalid character in Company Name '$_' at $invalid"
 			}
 		})]			
 		[string]$Company = "Huddled"		
 
 	)
 	end {
 		
 		$path = Join-Path ([Environment]::GetFolderPath("LocalApplicationData")) $Company
 		$path  = Join-Path $path $Module
 
 		if(!(Test-Path $path -PathType Container)) {
 			$null = New-Item $path -Type Directory -Force
 		}
 		$script:LocalStorageModuleName = $Module
 		Write-Output $path
 	}
 }
 
 function Export-LocalStorage {
 	#.Synopsis
 	#.Description
 	param(
 		[Parameter(Mandatory=$true, Position=0)]
 		[ValidateScript({ 
 			$invalid = $_.IndexOfAny([IO.Path]::GetInvalidFileNameChars())			
 			if($invalid -eq -1){ 
 				return $true
 			} else {
 				throw "Invalid character in Object Name '$_' at $invalid"
 			}
 		})]		
 		[string]$name,
 
 		[Parameter(Mandatory=$true, Position=1, ValueFromPipeline=$true)]
 		$InputObject,
 
 		[Parameter(Position=2)]
 		[ValidateScript({ 
 			$invalid = $_.IndexOfAny([IO.Path]::GetInvalidFileNameChars())			
 			if($invalid -eq -1){ 
 				return $true
 			} else {
 				throw "Invalid character in Module Name '$_' at $invalid"
 			}
 		})]		
 		[string]$Module = $LocalStorageModuleName
 	)
 	begin {
 		$path = Join-Path (Get-LocalStoragePath $Module) $Name
 		if($PSBoundParameters.ContainsKey("InputObject")) {
 			Write-Verbose "Clean Export"
 			Export-CliXml -Path $Path -InputObject $InputObject
 		} else {
 			$Output = @()
 		}
 	}
 	process {
 		$Output += $InputObject
 	}
 	end {
 		if($PSBoundParameters.ContainsKey("InputObject")) {
 			Write-Verbose "Tail Export"
 			if($Output.Count -eq 1) { $Output = $Output[0] }
 			Export-CliXml -Path $Path -InputObject $Output
 		}
 	}
 }
 
 function Import-LocalStorage {
 	#.Synopsis
 	#.Description
 	param(
 		[Parameter(Mandatory=$true, Position=0)]
 		[ValidateScript({ 
 			$invalid = $_.IndexOfAny([IO.Path]::GetInvalidFileNameChars())			
 			if($invalid -eq -1){ 
 				return $true
 			} else {
 				throw "Invalid character in Object Name '$_' at $invalid"
 			}
 		})]		
 		[string]$name,
 
 		[Parameter(Position=1)]
 		[ValidateScript({ 
 			$invalid = $_.IndexOfAny([IO.Path]::GetInvalidFileNameChars())			
 			if($invalid -eq -1){ 
 				return $true
 			} else {
 				throw "Invalid character in Module name '$_' at $invalid"
 			}
 		})]		
 		[string]$Module = $LocalStorageModuleName,
 
 		[Parameter(Position=2)]
 		[Object]$DefaultValue
 	)
 	begin {
 		if($PSBoundParameters.ContainsKey("Module")) {
 			$script:LocalStorageModuleName = $Module
 		}
 	}
 	end {
 		try {
 			$path = Join-Path (Get-LocalStoragePath $Module) $Name
 			Import-CliXml -Path $Path
 		} catch {
 			if($PSBoundParameters.ContainsKey("DefaultValue")) {
 				Write-Output $DefaultValue
 			} else {
 				throw
 			}
 		}
 	}
 }
 
 Export-ModuleMember -Function Import-LocalStorage, Export-LocalStorage, Get-LocalStoragePath -Variable LocalStorageModuleName
`

