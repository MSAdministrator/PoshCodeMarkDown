---
Author: brian
Publisher: 
Copyright: 
Email: 
Version: 1.0.0
Encoding: ascii
License: cc0
PoshCode ID: 4781
Published Date: 2016-01-08t14
Archived Date: 2016-10-25t20
---

# compare-folders.ps1 - 

## Description

this powershell script will compare all of the files in the same directory on two or more different computers. it will compute the md5 hash of a file and use that to compare

## Comments



## Usage



## TODO



## script

`reduce-set`

## Code

`#
 #
 <#
 .SYNOPSIS
 This PowerShell Script will compare all of the files in the same directory on two or more different computers. It will compute the MD5 hash of a file and use that to compare
 
 .DESCRIPTION
 Version - 1.0.0
 This PowerShell Script will compare all of the files in the same directory on two or more different computers
 
 .EXAMPLE
 .\Compare-Folders.ps1 -computers @("srv-1","srv-2") -path "D:\Inetpub" 
 
 .PARAMETER Computers
 Specifies the computers to compare. Mandatory parameter
 
 .PARAMETER Path
 Specifies the folder to check on the computers
 
 .PARAMETER csv
 Full Path to CSV file. If not passed, then results will display to the screen
 
 .NOTES
 
 #>
 
 [CmdletBinding(SupportsShouldProcess=$true)]
 param (
 	[Parameter(Mandatory=$true)] [string[]] $computers,
 	[Parameter(Mandatory=$true)] [string] $path, 
     [string] $csv
 )
 
 function Reduce-Set
 {
 	param(
 		[Parameter(ValueFromPipeline=$true)]
    		[HashTable] $ht
 	)
 	
 	begin { 
 		Set-Varible -Name differences -Values @()
 	}
 	process {		
 		Write-Verbose "Comparing Keys . . ."				
 		foreach ( $key in $ht.Keys ) {
 			if( $ht[$key].Count -eq 1 ) {		
 				$differences += (New-Object PSObject -Property @{
 					File = $ht[$key] | Select -ExpandProperty Name
 					System = $ht[$key] | Select -ExpandProperty System
 					Hash = $ht[$key] | Select -ExpandProperty FileHash
 				})
 			} 
 			elseif( ($ht[$key] | Select -Unique -ExpandProperty FileHash).Count -ne 1 )	{
 				foreach( $diff in $ht[$key] ) {
 					$differences += (New-Object PSObject -Property @{
 						File =  $diff.Name
 						System = $diff.System
 						Hash = $diff.FileHash
 					})
 				}
 			}
 		}
 		
 	}
 	end { 
 		return $differences
 	}
 }
 
 $map = {
 	param ( [string] $directory )
  	
 	function Get-Hash {
         param([string] $file)
 		$fileStream = [system.io.file]::openread($file)
 	    $hasher = [System.Security.Cryptography.HashAlgorithm]::create("md5")
 	    $hash = $hasher.ComputeHash($fileStream)
 	    $fileStream.Close()
 	    $md5 = ([system.bitconverter]::tostring($hash)).Replace("-","")
 	    return ( $md5 ) 
     }
 
     Set-Variable -Name files -Value @()
 	
 	Write-Verbose ("Working on - {0}" -f $ENV:COMPUTERNAME)
 
 	foreach( $file in (Get-ChildItem $directory -Recurse | Where { $_.PSIsContainer -eq $false } ) ) {
 		$files += New-Object PSObject -Property @{
             Name = $file.FullName
 			System = $ENV:COMPUTERNAME
             FileHash = (Get-Hash $file.FullName)
 		}
 	}
 	return $files
 } 
 
 function main
 {
 	$results = Invoke-Command -ComputerName $computers -ScriptBlock $map -ArgumentList $path | 
         Select Name, FileHash, System |
         Group-Object -Property Name -AsHashTable |
         Reduce-Set
     	
 	if( ![string]::IsNullOrEmpty($csv) ) {
 		$results | Export-Csv -Encoding Ascii -NoTypeInformation $csv
 		Invoke-Item $csv
 	}
 	else {
 		return $results
 	}
 }
 main
`

