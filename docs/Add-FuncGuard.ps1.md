---
Author: thell fowler
Publisher: 
Copyright: 
Email: 
Version: 0.0.1
Encoding: ascii
License: cc0
PoshCode ID: 1553
Published Date: 
Archived Date: 2009-12-25t16
---

#  - 

## Description

just a script in progress file 2

## Comments



## Usage



## TODO



## script

`add-funcguard`

## Code

`#
 #
 ####################################################################################################
 ##
 ##
 ####################################################################################################
 Set-StrictMode -Version 2
 
 function export {
     param (
         [parameter(mandatory=$true)] [validateset("function","variable")] $type,
         [parameter(mandatory=$true)] $name,
         [parameter(mandatory=$true)] $value
     )
 	if ( $args -ne "PSUnit" ) {
 		if ($type -eq "function")
 		{
 			Set-item "function:script:$name" $value
 			Export-ModuleMember $name
 		}
 		else
 		{
 			Set-Variable -scope Script $name $value
 			Export-ModuleMember -variable $name
 		}
 	}
 }
 
 export function Add-FuncGuard {
 	<#
 	.SYNOPSIS
 		Modifies the target C++ source code for a new function guard.
 	.DESCRIPTION
 		Function guards are a N++CR debug tool created by Jocelyn Legault for debug
 		function tracing.  Add-FuncGuard generates the required source code lines to
 		for function guard definition, enable/disable control, and importing.
 	.EXAMPLE
 		PS C:\NPPCR_ROOT>Add-FuncGuard -prefix "NPPCR" -path "$NPPCR_ROOT\PowerEditor\src\MISC\Debug\" `
 		>> -name "WinMain" `
 		>> -description "Trace functions in WinMain.cpp, should not be used in other files."
 	
 		Results in the following code being added:
 			... to FuncGuards_skel.h and FuncGuards.h (if it exists):
 				// func_guard(guard_WinMain);
 				// Description:  Trace functions in WinMain.cpp, should not be used in other files.
 	
 			... to FuncGuards.cpp:
 					func_guard_enable_cat(guard_WinMain);
 					func_guard_disable_cat(guard_WinMain);
 	
 			... to FuncGuardsImport.h
 				func_guard_import_cat(guard_WinMain);
 	
 	.PARAMETER Prefix
 		An all upper case string prefixing the preprocessor MACRO value.
 	.PARAMETER Path
 		Path to th function guard source files directory.
 	.PARAMETER Name
 		CamelCase_UnderscoreSpaced name of the function guard category.
 	.PARAMETER Description
 		Category notes inserted as comments to FuncGuards_skel.h && FuncGuards.h
 	#>
 	[CmdletBinding()]
 	param(
 		[Parameter(Position=0, Mandatory=$true)]
 		[string]
 		$Prefix = $(Throw 'Preprocessor MACRO prefix is required.')
 	,
 		[Parameter(Position=1, Mandatory=$true)]
 		[string]
 		$Path = $(Throw 'Path to required function guard source files is required.')
 	,
 		[Parameter(Position=2, Mandatory=$true)]
 		[string]
 		$Name = $(Throw 'CamelCase_UnderscoreSpaced function guard name is required.')
 	,
 		[Parameter(Position=3, Mandatory=$false)]
 		[string]
 		$Description
 	)
 	BEGIN {
 		try {
 			Test-PrefixParameter -Prefix $Prefix
 			Test-PathParameter -Path $Path
 			Test-NameParameter -Name $Name
 			Get-SourceFiles -sourceType "all" -Path $Path
 		}
 		catch {
 			Format-ErrorRecord $_ | Write-Debug
 			throw $_
 		}
 	}
 	PROCESS {
 		
 	}
 	END {
 		
 		
 		return $true
 	}
 }
 
 function Test-PrefixParameter {
 	param([string]$Prefix)
 }
 function Test-PathParameter {
 	param([string]$Path)
 	if (! (Test-Path -Path $Path -PathType Container) ) {
 		throw (New-Object IO.DirectoryNotFoundException $Path);
 	}
 }
 function Test-NameParameter {
 	param([string]$Name)
 	$retVal=$false
 	
 	if (! ( $Name -cmatch "(\p{Lu}{1}\p{Ll}+)+($|_((\p{Lu}{1}\p{Ll}+)|\d)+|\d+)+" ) ) {
 		throw (New-Object System.FormatException "Invalid function guard name format.")
 	}
 	
 	return $true
 }
 
 function Get-SourceFiles {
 	param([string]$sourceType,[string]$Path)
 	$SourceFiles = @{}
 
 	if ( ( $sourceType -eq "all" ) -or ( $sourceType -eq "required" ) ) {
 		$SourceFiles = (Get-RequiredFiles $Path)
 	}
 	return $SourceFiles
 }
 
 function Get-RequiredFiles {
 	param([string]$Path)
 	$RequiredSourceFiles = @{}
 
 	$TestFiles = @{
 		"SkeletonHeader" = (Join-Path -Path $Path -ChildPath "FuncGuards_skel.h");
 		"GuardSource" = (Join-Path -Path $Path -ChildPath "FuncGuards.cpp");
 		"GuardImport" = (Join-Path -Path $Path -ChildPath "FuncGuardsImport.h")
 	}
 	
 	$TestFiles.keys | foreach {
 		if (Test-Path -Path $TestFiles.$_ -PathType leaf) {
 			$RequiredSourceFiles.add("$_", (Get-Item $TestFiles.$_ ))
 		} else {
 			throw (New-Object IO.FileNotFoundException $TestFiles.$_)
 		}
 	}
 
 	$TestFiles = $null
 	
 	return $RequiredSourceFiles
 }
`

