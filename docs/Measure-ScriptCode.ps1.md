---
Author: victor vogelpoel
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4789
Published Date: 2014-01-13t07
Archived Date: 2014-01-15t19
---

# measure-scriptcode - 

## Description

measure-scriptcode calculates some code metrics like the number of lines-of-code, comments, functions from script or module files it is fed. the script is powershell 3 or later only because of use of ast (abstract syntax tree).

## Comments



## Usage



## TODO



## script

`measure-scriptcode`

## Code

`#
 #
 
 Set-PSDebug -Strict
 Set-StrictMode -Version Latest
 
 function Measure-ScriptCode
 {
 	[CmdletBinding()]
 	param
 	(
 	  [Parameter(Mandatory=$true, Position=1, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true, HelpMessage="One or more PS1 or PSM1 files to calculate code metrics for")]
 	  [Alias('PSPath', 'Path')]
 	  [string[]]$ScriptFile
 	)
 	
 	begin
 	{
 		$files 			= 0
 		$modules 		= 0
 		$manifests 		= 0
 	
 		$lines 			= 0
 		$words 			= 0
 		$characters		= 0
 		$codeLines 		= 0
 		$comments 		= 0
 		$functions		= 0
 		$workflows 		= 0
 		$filters 		= 0
 		$parseErrors	= 0
 	}
 
 	process
 	{
 		foreach ($file in $ScriptFile)
 		{
 			if ($file -like "*.ps1") { $files++ }
 			if ($file -like "*.psm1") { $modules++ }
 			if ($file -like "*.psd1") { $manifests++ }
 			
 			$fileContentsArray	= Get-Content -Path $file
 			
 			if ($fileContentsArray)
 			{
 				$measurement 		= $fileContentsArray | Measure-Object -Character -IgnoreWhiteSpace -Word -Line
 				$lines 			+= $measurement.Lines
 				$words 			+= $measurement.Words
 				$characters 		+= $measurement.Characters				
 		
 				$tokenAst			= $null
 				$parseErrorsAst		= $null
 				$scriptBlockAst		= [System.Management.Automation.Language.Parser]::ParseFile($file, [ref]$tokenAst, [ref]$parseErrorsAst)
 
 				$comments		+= @($tokenAst | where { $_.Kind -eq "Comment" } ).Length 
 				
 				$prevTokenIsNewline	= $false
 				$codeLines 		+= @($tokenAst | select -ExpandProperty Kind |  where { $_ -ne "comment" } | where {
 											if ($_ -ne "NewLine" -or (!$prevTokenIsNewline))
 											{
 												$_
 											}
 											$prevTokenIsNewline = ($_ -eq "NewLine")
 										} | where { $_ -eq "NewLine" }).Length-1
 				
 				$parseErrors 		+= @($parseErrorsAst).Length
 
 				if ($scriptBlockAst -ne $null)
 				{
 					$functionAst 	= $scriptBlockAst.FindAll({ $args[0] -is [System.Management.Automation.Language.FunctionDefinitionAst]}, $true)
 
 					$functions 	+= @($functionAst | where { (!$_.IsFilter) -and (!$_.IsWorkflow) }).Length
 					$filters	+= @($functionAst | where { $_.IsFilter }).Length
 					$workflows	+= @($functionAst | where { $_.IsWorkflow }).Length
 				}
 			}
 		}
 	}
 
 	end
 	{
 		return [PSCustomObject]@{
 			Files 		= $files
 			Modules 	= $modules
 			Manifests	= $manifests
 
 			CodeLines	= $codeLines
 			Comments 	= $comments
 			Functions 	= $functions
 			Workflows	= $workflows
 			Filters		= $filters
 
 			ParseErrors	= $parseErrors
 			Characters 	= $characters
 			Lines 		= $lines
 			Words		= $words
 		}
 	}
 }
`

