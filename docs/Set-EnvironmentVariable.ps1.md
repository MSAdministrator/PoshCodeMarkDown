---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1243
Published Date: 2009-07-29t09
Archived Date: 2012-07-30t02
---

# set-environmentvariable - 

## Description

allows you to easily create machine-level environment variables that will persist after reboots.  note that the variables created this way may not be visible in the variable

## Comments



## Usage



## TODO

needs v2 autohelp

## function

`set-environmentvariable`

## Code

`#
 #
 
 function Set-EnvironmentVariable {
 	param (
 		[String] [Parameter( Position = 0, Mandatory = $true )] $Name,
 		[String] [Parameter( Position = 1, Mandatory = $true )] $Value,
 		[EnvironmentVariableTarget] 
 			[Parameter( Position = 2 )]
 			$Target = [EnvironmentVariableTarget]::Process, 
 		[switch] $Passthru
 	)
 	[environment]::SetEnvironmentVariable( $Name, $Value, $Target )
 	if ( $Passthru ) {
 		$result = [environment]::GetEnvironmentVariable( $Name, $Target )
 		Write-Output @{ $Name = $Result }
 	}
 }
`

