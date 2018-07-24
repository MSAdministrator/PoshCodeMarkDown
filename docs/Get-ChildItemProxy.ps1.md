---
Author: andy schneider
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 785
Published Date: 2009-01-04t16
Archived Date: 2014-04-13t02
---

# get-childitemproxy - 

## Description

this is an advanced function that uses a proxy command to add two new switches to get-childitem

## Comments



## Usage



## TODO



## function

`get-childitemproxy`

## Code

`#
 #
 Function Get-ChildItemProxy {
 [CmdletBinding(DefaultParameterSetName='Items', SupportsTransactions=$true)]
 param(
     [Parameter(ParameterSetName='Items', Position=0, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.String[]]
     ${Path},
 
     [Parameter(ParameterSetName='LiteralItems', Mandatory=$true, Position=0, ValueFromPipelineByPropertyName=$true)]
     [Alias('PSPath')]
     [System.String[]]
     ${LiteralPath},
 
     [Parameter(Position=1)]
     [System.String]
     ${Filter},
 
     [System.String[]]
     ${Include},
 
     [System.String[]]
     ${Exclude},
 
     [Switch]
     ${Recurse},
 
     [Switch]
     ${Force},
 
     [Switch]
     ${Name},
     
     [Switch]
     ${ContainersOnly},
     
     [Switch]
     ${NoContainersOnly}
     )
 
 begin
 {
     try {
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer) -and $outBuffer -gt 1024)
         {
             $PSBoundParameters['OutBuffer'] = 1024
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Get-ChildItem', [System.Management.Automation.CommandTypes]::Cmdlet)
         
         if ($ContainersOnly)
         {
             [Void]$PSBoundParameters.Remove("ContainersOnly")
             $scriptCmd = {& $wrappedCmd @PSBoundParameters | Where-Object {$_.PSIsContainer -eq $true}}
             
         } elseif ($NoContainersOnly)
                {
                    [Void]$PSBoundParameters.Remove("NoContainersOnly")
                    $scriptCmd = {& $wrappedCmd @PSBoundParameters | Where-Object {$_.PSIsContainer -eq $false}}
                }    
         {
             $scriptCmd = {& $wrappedCmd @PSBoundParameters }
         }
         
 
         
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
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 <#
 
 .ForwardHelpTargetName Get-ChildItem
 .ForwardHelpCategory Cmdlet
 
 #>
 
 }
`

