---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3054
Published Date: 2012-11-19t15
Archived Date: 2012-02-05t03
---

# get-path - 

## Description

get-path converts relative paths to drive or psprovider -qualified paths.

## Comments



## Usage



## TODO



## function

`get-path`

## Code

`#
 #
 function Get-Path {
 [CmdletBinding(DefaultParameterSetName="DriveQualified")]
 Param(
    [Parameter(Position=0,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [Alias("PSPath")]
    [String]      
    $Path,
    [Parameter()]
    [Switch]$ResolvedPath,
    [Parameter(ParameterSetName="ProviderQualified")]
    [Switch]$ProviderQualified
    
 )
    $Drive = $Provider = $null 
    $PSPath = $PSCmdlet.SessionState.Path
    
    if($ResolvedPath -and $ProviderQualified) {
       $ProviderPaths = $PSPath.GetResolvedProviderPathFromPSPath($Path, [ref]$Provider)
    } else {
       $ProviderPaths = @($PSPath.GetUnresolvedProviderPathFromPSPath($Path, [ref]$Provider, [ref]$Drive))
       if($ResolvedPath) {
          $ProviderPaths = $PSPath.GetResolvedProviderPathFromProviderPath($ProviderPaths[0], $Provider)
       }
    }
    
    foreach($p in $ProviderPaths) {
       if($ProviderQualified -or ($Drive -eq $null)) {
          if(!$PSPath.IsProviderQualified($p)) {
             $p = $Provider.Name + '::' + $p
          }
       } else {
          if($PSPath.IsProviderQualified($p)) {
             $p = $p -replace [regex]::Escape( ($Provider.Name + "::") )
          }
          $p = $p.Replace($Drive.Root, $Drive.Name + ":\")
       } 
       $p
    }
 }
`

