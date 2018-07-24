---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5329
Published Date: 2016-07-25t21
Archived Date: 2016-05-19t08
---

# clipboard.psm1 - 

## Description

functions to copy and paste to the clipboard.

## Comments



## Usage



## TODO



## module

`get-clipboard`

## Code

`#
 #
 <#
 
 This module contains 2 functions:
 
 
 and a smart "clip" function that defers to Get-Clipboard or Out-Clipboard depending on when it's used as input or output, i.e.:
 
 clip |% ToUpper | clip
 
 becomes:
 
 Get-Clipboard |% ToUpper | Out-Clipboard
 
 #>
 
 Set-StrictMode -Version Latest
 
 Add-Type -Assembly PresentationCore
 
 function Get-Clipboard {
 	[CmdletBinding()]
 	param(
 		[switch]$Raw
 	)
 	$ret = [Windows.Clipboard]::GetText()
 	if ($Raw) {
 		$ret
 	} else {
 		$ret -split '\r?\n'
 	}
 }
 
 function Out-Clipboard {
 	[CmdletBinding(DefaultParameterSetName='S')]
 	param(
 		[AllowEmptyString()]
 		[Parameter(Mandatory, Position=0, ValueFromPipeline)]
 		[string]$InputObject
 ,
 		[Parameter(ParameterSetName='S')]
 		[string]$Separator = [System.Environment]::NewLine
 ,
 		[Parameter(ParameterSetName='T')]
 		[string]$Terminator = $Separator
 	)
 	begin {
 		$sb = [System.Text.StringBuilder]''
 	}
 	process {
 		$null = $sb.Append($InputObject)
 		$null = $sb.Append($Terminator)
 	}
 	end {
 		if ('S' -ceq $PSCmdlet.ParameterSetName -and 0 -lt $sb.Length) {
 			$sb.Length -= $Terminator.Length
 		}
 		[Windows.Clipboard]::SetText( ($sb.ToString() -replace '(?<!\r)\n',[System.Environment]::NewLine) )
 	}
 }
 
 function clip {
 	[CmdletBinding()]
 	param(
 		[AllowEmptyString()]
 		[Parameter(Mandatory, Position=0, ValueFromPipeline, ParameterSetName='Out-Clipboard')]
 		[string]$InputObject
 ,
 		[Parameter(ParameterSetName='Out-Clipboard')]
 		[string]$Separator = [System.Environment]::NewLine
 ,
 		[Parameter(ParameterSetName='Get-Clipboard')]
 		[switch]$Raw
 	)
 	begin {
 		try {
 			if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$null)) { $PSBoundParameters['OutBuffer'] = 1 }
 			$wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand($PSCmdlet.ParameterSetName, [System.Management.Automation.CommandTypes]::Function)
 			$steppablePipeline = {&$wrappedCmd @PSBoundParameters}.GetSteppablePipeline($MyInvocation.CommandOrigin)
 			$steppablePipeline.Begin($PSCmdlet)
 		} catch {
 			throw
 		}
 	}
 	process { try { $steppablePipeline.Process($_) } catch { throw } }
 	end { try { $steppablePipeline.End() } catch { throw } }
 }
`

