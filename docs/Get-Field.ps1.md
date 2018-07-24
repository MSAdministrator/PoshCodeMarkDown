---
Author: andrew savinykh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2057
Published Date: 2011-08-10t05
Archived Date: 2016-01-25t19
---

# get-field - 

## Description

displays private fields of passed object. this function is useful only if you want to want to see what is under the hood in powershell

## Comments



## Usage



## TODO



## function

`get-field`

## Code

`#
 #
 function Get-Field{
 [CmdletBinding()]
 	param ( 
 		[Parameter(Position=0,Mandatory=$true)]
 		$InputObject
 	)
 	
 	$type = $InputObject.gettype()
 	
 	$publicNonPublic = [Reflection.BindingFlags]::Public -bor [Reflection.BindingFlags]::NonPublic
 	$instance = $publicNonPublic -bor [Reflection.BindingFlags]::Instance
 	$getField = $instance -bor [Reflection.BindingFlags]::GetField
 	
 	$fields = $type.GetFields($instance)
 	
 	$result = @{}
 	$fields | Foreach-Object { $result[$_.Name] =  $type.InvokeMember($_.Name, $getField, $null, $InputObject, $null) } 
 	
 	$result
 	
 }
 
 ##$context = (Get-Field $ExecutionContext)._context
 ##$context
 ##$sessionState = (Get-Field $context)._enginesessionstate
 ##$sessionState
 ##$moduleTable = (Get-Field $sessionState)._moduleTable
 ##$moduleTable
`

