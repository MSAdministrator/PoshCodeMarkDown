---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1210
Published Date: 
Archived Date: 2009-07-22t11
---

#  - 

## Description

problem

## Comments



## Usage



## TODO



## function

`out-default`

## Code

`#
 #
 function out-default() {
 [CmdletBinding()]
 param(
     [Parameter(ValueFromPipeline=$true)]
     [System.Management.Automation.PSObject]
     ${InputObject})
 
 begin
 {
     
     try {
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer) -and $outBuffer -gt 1024)
         {
             $PSBoundParameters['OutBuffer'] = 1024
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Out-Default', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         $steppablePipeline = $scriptCmd.GetSteppablePipeline()
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         $steppablePipeline.Process($_)
         if ($last_output -eq $null)
         {
             $last_output = @()
         }
         if ($last_output.Length -lt 10)
         {
             $last_output += $_
         }
         else
         {
             $null, $last_output = $last_output
             $last_output += $_
         }
         
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
         $global:last_output = $last_output
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Out-Default
 .ForwardHelpCategory Cmdlet
 
 #>
 
 }
`

