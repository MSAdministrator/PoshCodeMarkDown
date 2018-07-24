---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 1000.0
Encoding: ascii
License: cc0
PoshCode ID: 5718
Published Date: 2016-01-28t05
Archived Date: 2016-05-19t09
---

# register-timer - 

## Description

the equivalent of settimer/setinterval/settimeout in powershell.

## Comments



## Usage



## TODO



## function

`register-timer`

## Code

`#
 #
 function Register-Timer {
 	[CmdletBinding()]
 	param(
 		[Parameter(Mandatory, Position = 0)]
 		[ValidateRange(0.00050000000000000012, 2147483.6474999995)]
 		[double]$TimeoutSec
 ,
 		[Parameter(Mandatory, Position = 1)]
 		[scriptblock]$Action
 ,
 		[string]$SourceIdentifier
 ,
 #.Parameter SupportEvent
 		[switch]$SupportEvent
 ,
 #.Parameter MessageData
 		$MessageData = $null
 ,
 		[int]$Count = 1
 	)
 	$t = [Timers.Timer][int](1000.0*$TimeoutSec)
 	$extra = @{}
 	if ($PSBoundParameters.ContainsKey('SourceIdentifier')) {
 		$extra['SourceIdentifier'] = $SourceIdentifier
 	}
 	$ret = Register-ObjectEvent -InputObject $t -EventName Elapsed -Action $Action -MessageData $MessageData -MaxTriggerCount $Count -SupportEvent: $SupportEvent @extra
 	$t.Start()
 	$ret
 }
`

