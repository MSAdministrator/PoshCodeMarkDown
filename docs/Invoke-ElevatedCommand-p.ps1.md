---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2297
Published Date: 2011-10-13t16
Archived Date: 2016-05-17t16
---

# invoke-elevatedcommand.p - 

## Description

enhanced error handling

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param
 (
     [Parameter(Mandatory = $true)]
     [ScriptBlock] $Scriptblock,
  
     [Parameter(ValueFromPipeline = $true)]
     $InputObject,
  
     [switch] $EnableProfile
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
  
     $commandString = "Set-Location '$($pwd.Path)'; " +
         "`$output = Import-CliXml '$inputFile' | " +
         "& {" + $scriptblock.ToString() + "} 2>&1 ; " +
 		"Out-File -filepath `$errorFile -inputobject `$error;" +
 		"Export-CliXml -Depth 1 -In `$output '$outputFile';"
  
     $commandBytes = [System.Text.Encoding]::Unicode.GetBytes($commandString)
     $encodedCommand = [Convert]::ToBase64String($commandBytes)
     $commandLine += "-EncodedCommand $encodedCommand"
  
     $process = Start-Process -FilePath (Get-Command powershell).Definition `
         -ArgumentList $commandLine -Verb RunAs `
         -Passthru `
         -WindowStyle Hidden
     $process.WaitForExit()
 	
 	if($process.ExitCode -eq 0)
 	{
 		if((Get-Item $outputFile).Length -gt 0)
 		{
 			Import-CliXml $outputFile
 		}
 	}
 	else
 	{
 		Write-Error -Message $(gc $errorFile | Out-String)
 	}
 	
     Remove-Item $outputFile
     Remove-Item $inputFile
     Remove-Item $errorFile
 }
`

