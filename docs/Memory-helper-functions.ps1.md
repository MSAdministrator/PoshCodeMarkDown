---
Author: steven murawski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 973
Published Date: 2009-03-25t18
Archived Date: 2012-11-05t14
---

# memory helper functions - 

## Description

memory management helpers.  set-sessionvariablelist creates a baseline of variable names.  remove-newvariable clears out any variables not in the sessionvariablelist and calls the garbage collector to free up memory.

## Comments



## Usage



## TODO



## function

`add-sessionvariable`

## Code

`#
 #
 Function Add-SessionVariable()
 {
 	param ([string[]]$VariableName=$null)
 	
 	[string[]]$VariableNames = [AppDomain]::CurrentDomain.GetData('Variable')
 	$VariableNames += $VariableName
 	
 	if ($input)
 	{
 		$VariableNames += $input
 	}
 	
 	$VariableNames = $VariableNames | Select-Object -Unique
 	
 	[AppDomain]::CurrentDomain.SetData('Variable',$VariableNames)
 }
 
 Function Set-SessionVariableList()
 {
 	$VariableNames = Get-Variable -scope global| ForEach-Object {$_.Name}
 	Add-SessionVariable $VariableNames
 	
 	Write-Verbose 'Loaded Variable Names into AppDomain'
 	$counter = 1
 	Foreach ($Variable in $VariableNames)
 	{
 		Write-Verbose "`t $($counter): $Variable" 
 		$counter++
 	}
 }
 
 Function Get-SessionVariableList()
 {
 	[AppDomain]::CurrentDomain.GetData('Variable')
 }
 
 Function Remove-NewVariable()
 {
 	$StartingMemory = (Get-Process -ID $PID).WS / 1MB
 	Write-Verbose "Current Memory Usage: $StartingMemory MB"
 
 	$VariableNames = Get-SessionVariableList
 	$VariableNames += 'StartingMemory'
 	Get-Variable -scope global | Where-Object {$VariableNames -notcontains $_.name} | Remove-Variable -scope global
 	
 	[GC]::Collect()
 	
 	$EndingMemory = (Get-Process -ID $PID).WS / 1MB
 	Write-Verbose "Ending Memory: $EndingMemory MB"
 	
 	$Diff = $StartingMemory - $EndingMemory
 	Write-Verbose "Freed up: $Diff MB"
 }
`

