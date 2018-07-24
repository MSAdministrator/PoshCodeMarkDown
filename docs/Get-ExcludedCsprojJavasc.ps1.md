---
Author: george mauer
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3821
Published Date: 2012-12-13t07
Archived Date: 2012-12-16t04
---

# get-excludedcsprojjavasc - 

## Description

do you create javascripts outside of visual studio and are constantly forgetting to include them? does that break your deployments or automated build? this script will check your csproj file and your scripts directory and let you know which javascripts are missing from your csproj.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
     $projectDirectoryName = "MyProject"
     )
 $thisDir = Split-Path $MyInvocation.MyCommand.Path
 $projectDir = "$thisDir/../$projectDirectoryName"
 
 $csproj = [xml](cat $projectDir/*.csproj)
 $csprojScripts = $csproj.Project.ItemGroup.Content.Include | ? {$_ -match '\.js$'}
 "$($csprojScripts.length) scripts included in csproj file"
 
 $scripts = ls $projectDir -rec -inc *.js
 $scripts = $scripts.FullName | % {($_ -match "$projectDirectoryName\\(.*)" | out-null); $matches[1]}
 "$($scripts.length) scripts contained in scripts directory"
 
 $diff = $scripts |? {-not ($csprojScripts -contains $_)}
 "Scripts in directory but not in csproj:"
 $diff
 
 return $diff.length
`

