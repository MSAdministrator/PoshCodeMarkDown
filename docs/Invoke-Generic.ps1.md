---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4638
Published Date: 2013-11-25t10
Archived Date: 2013-11-27t10
---

# invoke-generic - 

## Description

invoke generic method definitions (including static methods) from powershell.

## Comments



## Usage



## TODO



## function

`invoke-generic`

## Code

`#
 #
 function Invoke-Generic {
 #.Synopsis
 [CmdletBinding()]
 param( 
    [Parameter(Position=0,Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
    [Alias('On','Type')]
    $InputObject
 ,
    [Parameter(Position=1,ValueFromPipelineByPropertyName=$true)]
    [Alias('Named')]
    [string]$MethodName
 ,
    [Parameter(Position=2)]
    [Alias('Returns')]
    [Type]$ReturnType
 , 
    [Parameter(Position=3, ValueFromRemainingArguments=$true, ValueFromPipelineByPropertyName=$true)]
    [Object[]]$WithArgs
 )
 process {
    $Type = $InputObject -as [Type]
    if(!$Type) { $Type = $InputObject.GetType() }
    
    [Type[]]$ArgumentTypes = $withArgs | % { $_.GetType() }   
    [Object[]]$Arguments = $withArgs | %{ $_.PSObject.BaseObject }
    
    $Type.GetMethod($MethodName, $ArgumentTypes).MakeGenericMethod($returnType).Invoke( $on, $Arguments )
 } }
`

