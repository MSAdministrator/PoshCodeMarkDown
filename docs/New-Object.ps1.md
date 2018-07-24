---
Author: matt graeber
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4193
Published Date: 2016-06-06t09
Archived Date: 2016-04-08t09
---

# new-object - 

## Description

proxy function for new-object that allows you to specify a clsid in the comobject parameter. example

## Comments



## Usage



## TODO



## function

`new-object`

## Code

`#
 #
 function New-Object
 {
     [CmdletBinding(DefaultParameterSetName='Net', HelpUri='http://go.microsoft.com/fwlink/?LinkID=113355')]
     param(
         [Parameter(ParameterSetName='Net', Mandatory=$true, Position=0)]
         [string]
         ${TypeName},
 
         [Parameter(ParameterSetName='Com', Mandatory=$true, Position=0)]
         [string]
         ${ComObject},
 
         [Parameter(ParameterSetName='Net', Position=1)]
         [Alias('Args')]
         [System.Object[]]
         ${ArgumentList},
 
         [Parameter(ParameterSetName='Com')]
         [switch]
         ${Strict},
 
         [System.Collections.IDictionary]
         ${Property})
 
     begin
     {
         try {
             $outBuffer = $null
             if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
             {
                 $PSBoundParameters['OutBuffer'] = 1
             }
 
             $ClsidPresent = $false
             $Guid = [Guid]::NewGuid()
             if ([Guid]::TryParse($PSBoundParameters['ComObject'], [ref]$Guid))
             {
                 $ClsidPresent = $true
             }
             else
             {
                 $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('New-Object', [System.Management.Automation.CommandTypes]::Cmdlet)
                 $scriptCmd = {& $wrappedCmd @PSBoundParameters }
                 $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
                 $steppablePipeline.Begin($PSCmdlet)
             }
         } catch {
             throw
         }
     }
 
     process
     {
         if ($ClsidPresent)
         {
             [Activator]::CreateInstance([Type]::GetTypeFromCLSID($Guid), $Property)
         }
         else
         {
             try {
                 $steppablePipeline.Process($_)
             } catch {
                 throw
             }
         }
     }
 
     end
     {
         if (!$ClsidPresent)
         {
             try {
                 $steppablePipeline.End()
             } catch {
                 throw
             }
         }
     }
     <#
 
     .ForwardHelpTargetName New-Object
     .ForwardHelpCategory Cmdlet
 
     #>
 }
`

