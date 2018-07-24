---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2175
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t23
---

# invoke-binaryprocess.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Invokes a process that emits or consumes binary data.
 
 .EXAMPLE
 
 Invoke-BinaryProcess binaryProcess.exe -RedirectOutput |
        Invoke-BinaryProcess binaryProcess.exe -RedirectInput
 
 #>
 
 param(
     [string] $ProcessName,
 
     [Alias("Input")]
     [switch] $RedirectInput,
 
     [Alias("Output")]
     [switch] $RedirectOutput,
 
     [string] $ArgumentList
 )
 
 Set-StrictMode -Version Latest
 
 $processStartInfo = New-Object System.Diagnostics.ProcessStartInfo
 $processStartInfo.FileName = (Get-Command $processname).Definition
 $processStartInfo.WorkingDirectory = (Get-Location).Path
 if($argumentList) { $processStartInfo.Arguments = $argumentList }
 $processStartInfo.UseShellExecute = $false
 
 $processStartInfo.RedirectStandardOutput = $true
 $processStartInfo.RedirectStandardInput = $true
 
 $process = [System.Diagnostics.Process]::Start($processStartInfo)
 
 if($redirectInput)
 {
     $inputBytes = @($input)
     $process.StandardInput.BaseStream.Write($inputBytes, 0, $inputBytes.Count)
     $process.StandardInput.Close()
 }
 else
 {
     $input | % { $process.StandardInput.WriteLine($_) }
     $process.StandardInput.Close()
 }
 
 if($redirectOutput)
 {
     $byteRead = -1
     do
     {
         $byteRead = $process.StandardOutput.BaseStream.ReadByte()
         if($byteRead -ge 0) { $byteRead }
     } while($byteRead -ge 0)
 }
 else
 {
     $process.StandardOutput.ReadToEnd()
 }
`

