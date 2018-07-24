---
Author: rich kusak
Publisher: 
Copyright: 
Email: 
Version: 1.0.1.0
Encoding: ascii
License: cc0
PoshCode ID: 3075
Published Date: 2011-11-30t08
Archived Date: 2011-12-04t06
---

# new-pemfile - 

## Description

the new-pemfile function creates a new pem file by using the content from certificate files.

## Comments



## Usage



## TODO



## function

`new-pemfile`

## Code

`#
 #
 function New-PemFile {
 <#
 	.SYNOPSIS
 		Creates a new PEM file from one or more certificate files.
 
 	.DESCRIPTION
 		The New-PemFile function creates a new PEM file by using the content from certificate files.
 		CER, CRT, and KEY files are the most common certificate file types.
 
 	.PARAMETER Path
 		The path or paths to certificate files from which to create the PEM file.
 
 	.PARAMETER Target
 		Specifies the full path to the target PEM file.
 
 	.EXAMPLE
 		New-PemFile -Path C:\temp\domain.crt -Target C:\temp\domain.pem
 		Creates a PEM file from a single CRT file.
 
 	.EXAMPLE
 		New-PemFile -Path C:\temp\domain.crt, C:\temp\intermediate.crt, C:\temp\root.crt -Target C:\temp\domain.pem
 		Creates a PEM file from three CRT files.
 		
 	.EXAMPLE
 		$files = dir C:\temp\*.crt
 		$files[2,0,1] | New-PemFile -Target C:\temp\domain.pem
 		Captures all the CRT files in the temp folder, orders the files for input into the function, and creates the PEM file.
 		
 
 	.INPUTS
 		System.String
 
 	.OUTPUTS
 		None
 
 	.NOTES
 		Name: New-PemFile
 		Author: Rich Kusak
 		Created: 2011-11-05
 		LastEdit: 2011-11-30 09:46
 		Version: 1.0.1.0
 
 	.LINK
 		http://en.wikipedia.org/wiki/X.509
 
 	.LINK
 		http://www.digicert.com/ssl-support/pem-ssl-creation.htm
 
 #>
 
 	[CmdletBinding()]
 	param (
 		[Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
 		[ValidateScript({
 			foreach ($file in $_) {
 				if (Test-Path -LiteralPath $file) {$true} else {
 					throw "The argument '$_' is not a valid file."
 				}
 			}
 		})]
 		[Alias('FullName')]
 		[string[]]$Path,
 		
 		[Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
 		[ValidatePattern('\.pem$')]
 		[string]$Target
 	)
 	
 	begin {
 	
 		New-Item $Target -ItemType File -Force | Out-Null
 	}
 	
 	process {
 	
 		[PSObject[]]$paths += $Path
 	}
 
 	end {
 	
 		foreach ($file in $paths) {
 			Get-Content $file | Out-File $Target -Encoding ASCII -Append
 		}
 	}
`

