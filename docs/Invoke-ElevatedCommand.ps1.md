---
Author: obsidience
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3158
Published Date: 2012-01-09t01
Archived Date: 2016-05-17t13
---

# invoke-elevatedcommand - 

## Description

corrected the enhanced error handling, added window display option, corrected interactivity of hidden windows, added header area, wrapped in a function.

## Comments



## Usage



## TODO



## function

`invoke-elevatedcommand`

## Code

`#
 #
 Function Invoke-ElevatedCommand {
 	<#
 		.DESCRIPTION
 			Invokes the provided script block in a new elevated (Administrator) powershell process, 
 			while retaining access to the pipeline (pipe in and out). Please note, "Write-Host" output
 			will be LOST - only the object pipeline and errors are handled. In general, prefer 
 			"Write-Output" over "Write-Host" unless UI output is the only possible use of the information.
 			Also see Community Extensions "Invoke-Elevated"/"su"
 
 		.EXAMPLE
 			Invoke-ElevatedCommand {"Hello World"}
 
 		.EXAMPLE
 			"test" | Invoke-ElevatedCommand {$input | Out-File -filepath c:\test.txt}
 
 		.EXAMPLE
 			Invoke-ElevatedCommand {$one = 1; $zero = 0; $throwanerror = $one / $zero}
 
 		.PARAMETER Scriptblock
 			A script block to be executed with elevated priviledges.
 
 		.PARAMETER InputObject
 			An optional object (of any type) to be passed in to the scriptblock (available as $input)
 
 		.PARAMETER EnableProfile
 			A switch that enables powershell profile loading for the elevated command/block
 
 		.PARAMETER DisplayWindow
 			A switch that enables the display of the spawned process (for potential interaction)
 
 		.SYNOPSIS
 			Invoke a powershell script block as Administrator
 
 		.NOTES
 			Originally from "Windows PowerShell Cookbook" (O'Reilly), by Lee Holmes, September 2010
 				at http://poshcode.org/2179
 			Modified by obsidience for enhanced error-reporting, October 2010
 				at http://poshcode.org/2297
 			Modified by Tao Klerks for code formatting, enhanced/fixed error-reporting, and interaction/hanging options, January 2012
 				at https://gist.github.com/gists/1582185
 				and at http://poshcode.org/, followup to http://poshcode.org/2297
 			SEE ALSO: "Invoke-Elevated" (alias "su") in PSCX 2.0 - simpler "just fire" elevated process runner.
 	#>
 
 	param
 	(
 		[Parameter(Mandatory = $true)]
 		[ScriptBlock] $Scriptblock,
 	 
 		[Parameter(ValueFromPipeline = $true)]
 		$InputObject,
 	 
 		[switch] $EnableProfile,
 	 
 		[switch] $DisplayWindow
 	)
 	 
 	begin
 	{
 		Set-StrictMode -Version Latest
 		$inputItems = New-Object System.Collections.ArrayList
 	}
 	 
 	process
 	{
 		$null = $inputItems.Add($inputObject)
 	}
 	 
 	end
 	{
 
 		$outputFile = [IO.Path]::GetTempFileName()	
 		$inputFile = [IO.Path]::GetTempFileName()
 		$errorFile = [IO.Path]::GetTempFileName()
 
 		$inputItems.ToArray() | Export-CliXml -Depth 1 $inputFile
 	 
 		$commandLine = ""
 		if(-not $EnableProfile) { $commandLine += "-NoProfile " }
 
 		if(-not $DisplayWindow) { 
 			$commandLine += "-Noninteractive " 
 			$processWindowStyle = "Hidden" 
 		}
 		else {
 			$processWindowStyle = "Normal" 
 		}
 	 
 		$commandString = "Set-Location '$($pwd.Path)'; " +
 			"`$output = Import-CliXml '$inputFile' | " +
 			"& {" + $scriptblock.ToString() + "} 2>&1 ; " +
 			"Out-File -filepath '$errorFile' -inputobject `$error;" +
 			"Export-CliXml -Depth 1 -In `$output '$outputFile';"
 	 
 		$commandBytes = [System.Text.Encoding]::Unicode.GetBytes($commandString)
 		$encodedCommand = [Convert]::ToBase64String($commandBytes)
 		$commandLine += "-EncodedCommand $encodedCommand"
 
 		$process = Start-Process -FilePath (Get-Command powershell).Definition `
 			-ArgumentList $commandLine `
 			-Passthru `
 			-Verb RunAs `
 			-WindowStyle $processWindowStyle
 
 		$process.WaitForExit()
 
 		$errorMessage = $(gc $errorFile | Out-String)
 		if($errorMessage) {
 			Write-Error -Message $errorMessage
 		}
 		else {
 			if((Get-Item $outputFile).Length -gt 0)
 			{
 				Import-CliXml $outputFile
 			}
 		}
 
 		Remove-Item $outputFile
 		Remove-Item $inputFile
 		Remove-Item $errorFile
 	}
 }
`

