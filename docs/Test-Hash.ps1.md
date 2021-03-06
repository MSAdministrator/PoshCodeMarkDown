---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3414
Published Date: 2012-05-14t13
Archived Date: 2012-05-16t21
---

# test-hash - 

## Description

test md5/sha1/etc file hashes.

## Comments



## Usage



## TODO



## function

`test-hash`

## Code

`#
 #
 function Test-Hash { 
 #.Synopsis
 #.Description
 #.Example
 #
 #
 #.Example
 #
 #.Example
 #
 #
 #.Example
 #
 [CmdletBinding(DefaultParameterSetName="NoExpectation")]
 PARAM(
    [Parameter(Position=0,Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("PSPath")]
    [string]$Path,
    
    [Parameter(Position=2,Mandatory=$false,ParameterSetName="NoExpectation")]
    [string]$BasePath,
    
    [Parameter(Position=1,Mandatory=$true,ParameterSetName="FromHashFile")]
    [string]$HashFileName,
    
    [Parameter(Position=2,Mandatory=$true,ParameterSetName="ManualHash", ValueFromPipelineByPropertyName=$true)]
    [Parameter(Position=2,Mandatory=$false,ParameterSetName="FromHashFile")]
    [ALias("Hash")]
    [string]$ExpectedHash = $(if($HashFileName){  ((Get-Content $HashFileName) -match $Path)[0].split(" ")[0]  }),
    
    [Parameter(Position=1,Mandatory=$true,ParameterSetName="ManualHash", ValueFromPipelineByPropertyName=$true)]
    [Parameter(Position=1,Mandatory=$false,ParameterSetName="NoExpectation")]
    [ValidateSet("MD5","SHA1","SHA256","SHA384","SHA512","RIPEMD160")]
    [string[]]$Algorithm = $(if($HashFileName){ [IO.Path]::GetExtension((Convert-Path $HashFileName)).Substring(1) } else { "SHA256" })
 )
 
 begin {
    $ofs=""
    if($BasePath) {
       Push-Location $BasePath
    }
 }  
 
 process {
    if($BasePath) {
       $Path = Resolve-Path $Path -Relative
    }
    if(Test-Path $Path -Type Container) {
       Write-Warning "Cannot calculate hash for directories: '$Path'"
       return
    }
 
    $Hashes = @(
       foreach($Type in $Algorithm) {
          [string]$hash = [Security.Cryptography.HashAlgorithm]::Create( $Type ).ComputeHash( [IO.File]::ReadAllBytes( (Convert-Path $Path) ) ) | ForEach { "{0:x2}" -f $_ }
          New-Object PSObject -Property @{ Path = $Path; Algorithm = $Type; Hash = $Hash }
       }
    )
 
    if($ExpectedHash) {
       if( $Match = $Hashes | Where { $_.Hash -eq $ExpectedHash } ) {
          Write-Verbose "Matched hash:`n$($Match | Out-String)"
          New-Object PSObject -Property @{ Path = $Match.Path; Algorithm = $Match.Algorithm; Hash = $Match.Hash; Matches = $True }
       } else {
          Write-Verbose "Failed to match hash:`n$($PSBoundParameters | Out-String)"
          New-Object PSObject -Property @{ Path = $Hashes[0].Path; Algorithm = $Hashes[0].Algorithm; Hash = $Hashes[0].Hash; Matches = $False }
       }         
    } else {
       Write-Output $Hashes  
    }
 }
 end {  
    if($BasePath) {
       Pop-Location
    }
 }
 }
`

