---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 895
Published Date: 
Archived Date: 2010-05-15t14
---

# split-string - 

## Description

splits a string (by default, on whitespace), and allows you to pick and chose which pieces are returned. something like “cut” in bash…

## Comments



## Usage



## TODO



## function

`split-string`

## Code

`#
 #
 function Split-String {
 #.Synopsis
 #.Description
 #.Example
 #
 #
 #.Example
 #
 #
 #.Parameter pattern
 #.Parameter action
 #.Parameter InputObject
 [CmdletBinding(DefaultParameterSetName="DefaultSplit")]
 Param(
    [Parameter(Position=0, ParameterSetName="SpecifiedSplit")]
    [string]$pattern="\s+"
 ,
    [Parameter(Position=0,ParameterSetName="DefaultSplit")]
    [Parameter(Position=1,ParameterSetName="SpecifiedSplit")]
    [ScriptBlock]$action={$0}
 ,
    [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
    [string]$InputObject
 )
 BEGIN {
    if(!$pattern){[regex]$re="\s+"}else{[regex]$re=$pattern}
 }
 PROCESS {
    $0 = $re.Split($InputObject)
    $1,$2,$3,$4,$5,$6,$7,$8,$9,$n = $0
    &$action
 }
 }
    
  
`

